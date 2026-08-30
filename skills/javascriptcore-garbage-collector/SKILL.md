---
name: javascriptcore-garbage-collector
description: JSC GC 参考 for Bun. Use for use-after-free, JS 对象 leaks, \"collected too early\", or when touching WriteBarrier, visitChildren, visitAdditionalChildren, JSRef, JSC::Strong/Weak, hasPendingActivity, ensureStillAlive, addOpaqueRoot, reportExtraMemoryAllocated, IsIsoSubspaceHeaHeapAnalyzerinalize.
---

# JavaScriptCore's 垃圾回收器 (Riptide)

Riptide is **non-moving, generational, 并行, mostly-concurrent, conservconservative-on-the-stackerstanding those five words prevents most GC bugs in Bun.

## The mental 模型

The heap is a graph. GC does a breadth-first 搜索 from **roots** → marks everything it reaches → everything unmarked is freed (lazily, on next allocation from that block). It does NOT compact or move objects — pointers stay stable for an 对象's lifetime.

**Two collection modes:**

- **伊甸园 GC**: only scans newly-allocated objects + 记忆集. Fast, frequent.
- **Full GC**: scans everything. Slower, rarer.

**It runs concurrently.** Marking happens on background threads _while JS is executing_; the mutator only stops at brief safepoints. `visitChildren` runs **off the main 线程, racing with your code**.

## How the VM gathers roots

Roots are not a hardcoded 列表 — they are **marking constraints** registered with `Heap::addMarkingConstraint()` and 运行 迁移到 fixpoint. The built-in 集合 lives in `Heap::addCoreConstr`Heap::addCoreConstr`ints()`rce/JavaScriptCore/h`ve`ints()`PATH_1__DE_4__

| 标签   | Name              | What it marks                                                                                                                                                           |
| ----- | ----------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `Cs`  | Conservative Scan | Native stack + registers of every JS 线程, scan`gatherStackRoots`Roots`Roots`R`Roots`_COD`ConservativeRoots``ts` Also JIT stub routines. World is stopped for this.        |
| `Msr` | Misc `vm.smallStrings`ings`ings`i`ings`_C`m_protectedValues`s`ues``JSVa`J`JSVa`Pro`ue`ueProtect` `), `CODE`gcProtect``uf`Ma`MarkedArgument__CODE_`lastExcept``las`xcept`` / `xcept`lastException()`astException()`lastEx`ceptio`)`ceptio`m_terminationException`
| `Sh`  | St`m_handleSet.visitStrongHandles()`les()`les()`les()` — every `J`les()` — eve`JSC::St`Strong<T`JSC::Strong<T>`ggrega`. ``g`Aggregate()`tAggreg`vm().visitAggregate()`                      |
| `D`   | Debugger          | Sampling profiler, 类型 profiler, ShadowChicken                                                                                                                         |
| `Ws`  | Weak Sets        `WeakBlock`Block`B__COD`; c``; c``; c`WeakHandleOwner::isR`ts()``ts()`Opa`ts()`ts()`ts()` 迁移到 decide whether a weak ref 应该 _become_ strong this cycle                        |
| `O`   | 输`visitOutputConstraints()`ints()`ints()`ints()` `ints()`dy-marked cells in 输出-constraint subspaces (executables, WeakMaps). This is the "re-运行 after marking discovers more" hook |
| `Jw`  | JIT Worklist      | CodeBlocks queued for compilation                                                                                                                                       |
| `Cb`  | CodeBlocks        | Executing/compiling CodeBlocks                                                                                                                                          |

Bun registers an additional constraint, `DOMGCOutputConstraint` (`src/bun.js/b__CODE_1__ndings/BunGCOutp__CODE_2__s `aints` on every ` on ev`visitOutputCo` in B`C`aints`nts`ubs` on every marked cell in B`spaces (事件 targets, genera`ubspaces (event targets, gen`itionalChildren``, etc.).``.).``, etc.).`

**Constraint volatility** controls when they re-运行 during the fixpoint:

- `GreyedByExecution` — 可以 produce new grey cells whenever the mutator runs (re-运行 after every resume)
- `GreyedByMarking` — 可以 produce new grey cells when _other_ marking happens (re-运行 after each drain)
- `SeldomGreyed` — usually doesn't 添加 anything; 运行 last

## 对象 layout: the 8-byte JSCell header

Every GC-managed 对象 inherits `JSCell`CODE_1_`h`h`):

```
| StructureID (4) | indexingTypeAndMisc (1) | JSType (1) | flags (1) | cellState (1) |
```

- `StructureID` — compressed hidden-class pointer
- `indexingTypeAndMisc` — 2 bits are an embedded `WTF::锁``WTF::Lock`l`WTF::Lock`lw`WTF::Lock`s byte
- `cellState` — inlined GC color, used by the 写入 屏障

Out-of-line, in the `MarkedBlock` footer (or `Pr__CODE`ciseAllocation`on`ader for objects >~8KB):

- `isMarked` bit — survived last GC
- `isNewlyAllocated` bit — allocated since last GC

Liveness = `isMarked || isNewlyAllocated` (with logical-versioning so blocks aren't swept eagerly).

## CellState and the 写入 屏障

`vendor/WebKit/Source/JavaScriptCore/heap/CellState.h`:

```cpp
PossiblyBlack   = 0   // visited (or old-space-pending-rescan during full GC)
DefinitelyWhite = 1   // new / unmarked
PossiblyGrey    = 2   // on the mark stack
```

Generational + 并发垃圾回收 share **one** retreating-wavefront 屏障:

```cpp
// After: obj->field = newValue
if (obj->cellState <= blackThreshold)   // 0 normally, bumped while GC is marking
    writeBarrierSlowPath(obj);          // → put obj on remembered set / revisit
```

**You almost never 写入 this by hand.** Use `WriteBarrier<T>` as the 字段 类型 and call `.集合(v`.set(v`, ow`, owner, value)`tores then `tores then barriers. A raw `ue``JSCell*`it` / `lu` / `ell`JSCell*`u` / `CODE_7__`JSValue`C 将 free ` wrap`get out from under you.

`LazyProperty<Owner, T>`, `LazyClassStru`LazyClassStru`ture`rrierStructur`ture`rrierStructu`W`D`reID`nt`reID`lazily-initialized fields and structures.

## Allocation: where objects live

`bmalloc/libpas` provides pages; JSC carves them up:

- **`MarkedBlock`** — 16KB block, fixed cell size (segregated free 列表). Footer holds bitvectors. 16-byte minimum cell alignment. `ad__CODE`r & ~(16KB-1)`1)`block, so liveness checks are O(1).
- **`PreciseAllocation`** — large objects (>~8KB), individually `malloc`'`malloc`_CODE_2____eader. Always 返回 addresses with `addr`addr`%``% 16 == 8` `r`r & 8`CODE_6__`s them from MarkedBlock cells.
- **`CompleteSubspace`** — size-segregated 集合 of `BlockDi`BlockDi`ectory`ectory`S objects.
- **`IsoSubspace`** — one 子空间 per C++ 类型 (security: a freed cell 可以 only be reused for the _same_ 类型, defeating type-confusion UAF). **Every Bun 类 with native fields needs its own IsoSubspace** — `su__CODE`spaceFor`or` the header, 槽 `n `n `BunClient`unClient`BunCl`IsoSubspace`ubspace`DOMIsoSubspaces`

**Allocation 可以 trigger GC.** A safepoint exists at every allocation. Never assume "I just allocated X, so Y from before is still alive" unless Y is rooted.

## Conservative stack 扫描 — 功能 and doesn't guarantee

`vendor/WebKit/Source/JavaScriptCore/heap/ConservativeRoots.cpp` walks the native stack/registers word-by-word (after `MachineThreads::tryCopyOtherThreadStacks` snapshots t`MachineThreads::tryCopyOtherThreadStacks`edBlock` cell or `PrecPreciseAllocation a ro`MachineThreads::tryCopyOtherThreadStacks`` cell or `` is a root.``PreciseAllocat` cell or `` is a ro`` cell or `` is a root.``PreciseAllocation`

**This means:** a `JSCell*``JSValue``_CODE_2_`处理`理`_COD` ceremony like V8.`ike V8.`_CODE_3__cal`

**This does NOT mean you're always safe.** The compiler 可以 dead-store-eliminate the local after its last visible use, or never spill it. If you 提取 an interior pointer (`string->characters8()`, butterfly 存储, typed-arratyped-array`) a`vector()`ll `vector()`tha`vector()`ate, the original cell 可以 no longer be on the stack:

```cpp
JSC::EnsureStillAliveScope keepAlive(cell);   // RAII: forces cell onto stack until scope end
// ... use interior pointer, call things that allocate ...
```

or `ensureStillAliveHere(cell)`. In Zig: `值.ensureStill`value.ensureStill`live()``live()`

## `visitChildren` — the per-cell 追踪 hook

```cpp
// In header:
DECLARE_VISIT_CHILDREN;
WriteBarrier<JSObject> m_callback;
WriteBarrier<Unknown>  m_cachedValue;

// In .cpp:
template<typename Visitor>
void JSFoo::visitChildrenImpl(JSCell* cell, Visitor& visitor) {
    auto* thisObject = jsCast<JSFoo*>(cell);
    ASSERT_GC_OBJECT_INHERITS(thisObject, info());
    Base::visitChildren(thisObject, visitor);   // ALWAYS call base first

    visitor.append(thisObject->m_callback);
    visitor.append(thisObject->m_cachedValue);
}
DEFINE_VISIT_CHILDREN(JSFoo);
```

**Rules — runs concurrently on a GC 线程:**

- No allocation. No __CODE_`jsString`ing`, nothing that touc`ouc`vm.heap`ea`vm.heap``.`
- No `ref()`CODE_`RefCounted`ed`e`ted`` (not thread-safe).
- No locks the main 线程 也许 also take while allocating (死锁).
- If a 字段 可以 be torn by a racing mutator, take `Locker locker { thisObject->cellLock() }` in both `visitChildren` and the mutating`visitChildren``visitChildren``visitChildren``visitChildren`
- Forgetting 迁移到 `append()` a `WriteBarrier` 字段 → use-after-free, often 伊甸园-GC-only, often only under load.

## `visitAdditionalChildren` and 输出 constraints

`visitChildren` only sees the cell's own fields. When a JS wrapper's liveness 应该 propagate 迁移到 **other JS objects reachable through native state** (事件 listeners, observers, the JS values held inside a wrapped C++ 对象), Bun uses the WebCore pattern:

```cpp
// Custom hook called from BOTH places:
template<typename Visitor>
void JSFoo::visitAdditionalChildren(Visitor& visitor) {
    wrapped().listeners().visitJSEventListeners(visitor);
    visitor.addOpaqueRoot(&wrapped());
}

// 1) From visitChildren (normal marking):
DEFINE_VISIT_CHILDREN_WITH_MODIFIER(..., JSFoo) {
    ...
    thisObject->visitAdditionalChildren(visitor);
}

// 2) From visitOutputConstraints (constraint fixpoint re-scan):
template<typename Visitor>
void JSFoo::visitOutputConstraints(JSCell* cell, Visitor& visitor) {
    auto* thisObject = jsCast<JSFoo*>(cell);
    Base::visitOutputConstraints(thisObject, visitor);
    thisObject->visitAdditionalChildren(visitor);
}
```

**Why two entry points?** `visitChildren` runs once when the cell turns grey. But marking 可以 later discover that some _other_ native 对象 (an opaque root) is live, which retroactively makes more of _this_ cell's 参考 live. `visi`visi``OutputConstraints`e-invoked`e-invoked by `Constra`DOMGC`DOMGCOut`DOMGC`traint`xpoint 迁移到 catch that.

迁移到 make a 类 participate, its IsoSubspace 必须 be registered as an **输出-constraint 子空间** (`clientSubspaceFor*` with `outputCon`outputCon`traint`ien`traint`ientDa` `n`ntD`nClientD`sses.cp`cp`ZigGenerate`ZigGeneratedClasses.cpp`cpp`when `.classes.t`.classes.ts`ndingAc``hasPendingAc`s.ts`r`hasPendi`ses.ts`_CODE_10__ses.ts``hasPend`tic`tivity`hasPendingActivity``own`

## Opaque roots — liveness through non-JSCell pointers

When native objects form a graph that 应该 keep wrappers alive:

```cpp
// In some wrapper's visitAdditionalChildren:
visitor.addOpaqueRoot(nativePtr);          // "nativePtr is reachable"

// Elsewhere, deciding whether ANOTHER wrapper survives:
bool JSBarOwner::isReachableFromOpaqueRoots(Handle<Unknown> h, void* ctx,
                                            AbstractSlotVisitor& v, ASCIILiteral* reason) {
    auto* bar = static_cast<Bar*>(ctx);
    if (UNLIKELY(reason)) *reason = "Bar is in document tree"_s;
    return v.containsOpaqueRoot(bar->ownerNode());
}
```

The opaque-root 集合 is just a `HashSet<void*>` rebuilt each cycle. It's how DOM trees stay alive as a 单位.

## `JSC::Weak<T>`, `Wea`Wea`CODE_2__`WeakHandleOwner`k`WeakHand`k`wner`

`JSC::Weak<T>` (`ven`ven`CODE_2__` the GC-aware weak pointer. It does **not** keep its target alive; ``.获取()` 返回 `nullptr` after the ta`.获取()`取`nullptr``nullptr`` 返回 ```.获取()`the`nullptr`_CODE` after the target``ected.`

Under the hood:

- Each `Weak<T>` owns`WeakImpl*```__CODE_2__vendor/WebKit/Source/JavaScriptCore/hea__CODE__CODE_4__`): `h`): `{ JSValue, Weak`{ JSValue, WeakHandleOwner* (low bits = state), void* context }`void* context }` context }`ocated`.`Live → Dead → Finalized → Dea`ocated`_CODE_7__`.`
- `WeakImpl`s are slab-allocated in 1KB **`WeakBlock`s** (````vendor/WebKit/Source/JavaScriptCore/heap/WeakBlock.h`blockSize = 1024`). Every `MarkedBlock` and `blockSize = 1024`` has a `WWeakSet`MarkedBlock`ist of ``blockSize = 1024``ls in t` has a ``WeakSe`MarkedBlock``PreciseAll`blockSize = 1024`WeakBloc`ls in t``WeakSe``PreciseAllocation``WeakSet``WeakBlock`
- During the `Ws``WeakBlock::visit()`sit()`sit()`sit`sit()`` walks its `eakIm`Wea`WeakImpl` whose target is **not yet marked**, it calls `Wea__CODE_`Wea``dleOwner::isReachableFromOpaqueRoots(处理, context, visitor, &r`t`CODE_8__n)`t`turn `u` → the target is marked (the weak ref is "upgrade`true`_CODE_10__w `hasPendingActivity()` and opaque-root rea`ow `lity`hasP`w `ppers aliv`hasP` and oopaque-rootreachability keep wrappers aliv`
- After marking, `WeakBlock::reap()` flips unmarked `Live` im`Live`_CODE_2___`__CODE`CODE_3__later runs `s `s `WeakHan`WeakHandle`WeakHandl`WeakHandleOwner::finalize(handle, context)`ees the 槽. **`finalize`D` impl, then frees the 槽. **`i`finalize`a` impl, then frees the slot. **`S fields.** Ty`finalize` drop`is already dead — do n`ve→JS wrapper cach`ch its JS fields.** Ty`

```cpp
struct MyOwner final : public JSC::WeakHandleOwner {
    bool isReachableFromOpaqueRoots(Handle<Unknown>, void* ctx,
                                    AbstractSlotVisitor& v, ASCIILiteral*) override {
        return static_cast<NativeThing*>(ctx)->hasPendingActivity();
    }
    void finalize(Handle<Unknown>, void* ctx) override {
        static_cast<NativeThing*>(ctx)->m_wrapper = nullptr;
    }
};
JSC::Weak<JSFoo> m_wrapper { jsFoo, &myOwnerSingleton, nativeThing };
```

`Weak<T>` is **move-only** (allocates`WeakImpl````). Don't put it in a hot 路径; 缓存 it.

## Zig: `jsc.JSRef` — the native↔wrapper 参考 pattern

In Bun's Zig code, when a native 对象 needs 迁移到 hold a 参考 back 迁移到 its own JS wrapper, **use `jsc.JSRef`** (````src/bun.js/bindings/JSRef.zig`not `gcProtect`, not a raw`gcProtect`field, and u`gcProtect``jsc.Strong` `gcProtect`j`jsc.Strong`sc.S`JSValue`_CODE_7__`jsc.Strong`

`JSRef` is a tagged union with three states:

- `.weak` `JSValue``. Does **not** keep the wrapper alive. Valid only because the wrappe`ppe`ppe`finalize()`e()` 将 flip this t`.finalized`d` before the `finalize()`e()`o` 将 flip this t`zed`` i` before `d` cell is freed, __CODE_`tryGet` _not`JSC` i`ak`eak`; it` 返回 `` inste`his is _not`ing pointer.`eak`CODE` instead of a dangling pointer. (This is _not``eak``WeakImpl`
- `.strong` — wra`jsc.Strong`````__CODE`SC::Strong<Unknown`n>``root). Keeps the wrapper alive.
- `.finalized` — terminal; `t__CO`y`1__CODE`null`ns ``null``null`

Pattern: **strong while busy, weak while idle.**

```zig
this_value: jsc.JSRef = .empty(),

// On construction / when work starts:
this.this_value.setStrong(js_wrapper, globalThis);   // or .upgrade(globalThis)

// When the last in-flight operation completes:
this.this_value.downgrade();                         // strong → weak, GC may now collect

// In any callback that needs the wrapper:
const js_this = this.this_value.tryGet() orelse return;

// In the codegen'd finalize():
this.this_value.finalize();
```

See `ServerWebSocket`, `UDPSoc`UDPSoc`et`L`et`CODE_3__`Conn`yClient`_CODE_5__Client``les.

**`JSRef` requires a finalize`.weak`ak`_CODE_2_`finalize()`ize()`i` flips it` it`.fi`ize()`CODE_5__` bef`` is r`zed` is r` before the cell is r__COD`finalize: true` ``finalize: true`(almo`s`CODE_10__ked`finali`JSRef`e`SRef``JSRef` default c__CODE_`JSRef`self-参考.`JSRef``JSRef`

**`JSRef`CODE_1_`ty``:** prefer ` prefer `_CO`` GC-thread-polled atomic predicate; its only real justification is when **many concurrent operations** independently keep the wrapper alive and there's no single place 迁移到 call ` 迁移到 call `升级()`/__CODE_`downgra`rade()`.`.,`wngra`le liveness where the count is touched from multiple threads. That's uncommon. If you 可以 identify "work started" / "work f`/__CODE_`dges, u`rade()`_CODE_7_`le liveness where the count is touched from multiple threads. That's uncommon. If you 可以 identify "work started" / "work finished" edges, use ``JSRef``hasP``ctivity``hasPendingActivity``. Don't 添加 ``JSRef`` `

## `gcProtect` / `````JSValueProtect`lmost never

`gcProtect()` / `JS__CODE`alueProtec`ec`sh into`o ```Heap::m_protectedValues`Values`es`nted ro`nted root map, visited by the `str`Msr`. It's th`Msr`ac`` constr`ch`. It's th`oid `ac`n Bun:**

- It's a raw 全局根 with manual unprotect — easy 迁移到 leak.
- It has no owner, so heap snapshots 可以't attribute the retention.
- `jsc.Strong` / `J__CO`Ref`_Ref`ive the same guarantee with RAII and a destructor.

The only legitimate uses are inside the JSC C API shims themselves, or one-off debugging.

## Extra-内存 reporting — `reportExtraMemoryAllocated` / `reportExtraMemory`reportExtraMemory`isited``isited``isited``isited`

The GC schedules itself by bytes-allocated-since-last-GC. It only sees JSCell allocations, so a 32-byte wrapper around a 50MB native 缓冲区 looks like 32 bytes → GC never triggers → OOM.

**Contract — both halves are required:**

```cpp
// 1) When the native memory is allocated (or the wrapper takes ownership):
vm.heap.reportExtraMemoryAllocated(ownerCell, byteCount);

// 2) In visitChildren, every time the cell is visited:
visitor.reportExtraMemoryVisited(thisObject->wrapped().byteSize());
```

- `reportExtraMemoryAllocated` adds 迁移到 the "since last GC" counter and 可以 **immediately trigger a GC** (it's a safepoint). Call it _after_ the cell is fully constructed.
- `reportExtraMemoryVisited` adds 迁移到 the "live bytes after this GC" counter, which sets the next trigger threshold. **If you forget this half**, the heap's high-water mark drifts down each cycle and you 获取 back-迁移到-back full GCs (the "GC death spiral").
- If the size changes over 时间, report the delta on growth (`reportExtraMemoryAllocated(cell, newSize - oldSize)`) and report the current size in `visitChildren`.`visitChildren``visitChildr`visitChildren`en``visitChildren``visitChildren``visitChildren`
- `deprecatedReportExtraMemory` exists for callers that 可以't satisfy the visit-side half — avoid it.

In `.classes.ts`, `es__CODE`imatedSize: t` t`nerates the `e`e `portExtr`reportExtraMemoryVisited`ted`ement `estimatedSi`estimatedSize()`ze()`im`estimatedSize()`M`estimatedSize()`o`tExtraM`portExtraMemor`or the ` the `时间.` (or the `

## `HeapAnalyzer` — heap snapshots and labelling

`vendor/WebKit/Source/JavaScriptCore/heap/HeapAnalyzer.h` is the abstract visitor used 迁移到 构建 heap snapshots (Web Inspector "Heap 快照", and Bun's V8-compatible `BunV8HeapSnapshotBuilder`). When a 快照 is`BunV8HeapSnapshotBuilder`ith an analyzer attached and each cel`BunV8HeapSnapshotBuilder`s called:`analyzeHeap``analyzeHeap``analyzeHeap``BunV8HeapSnapshotBuilder``analyzeHeap``analyzeHeap``analyzeHeap`

```cpp
void JSFoo::analyzeHeap(JSCell* cell, HeapAnalyzer& analyzer) {
    auto* thisObject = jsCast<JSFoo*>(cell);
    Base::analyzeHeap(cell, analyzer);
    analyzer.setWrappedObjectForCell(cell, &thisObject->wrapped());
    analyzer.setLabelForCell(cell, thisObject->wrapped().url().string());
    if (auto* child = thisObject->m_callback.get())
        analyzer.analyzePropertyNameEdge(cell, child, vm.propertyNames->callback.impl());
}
```

API (`HeapAnalyzer`):

- `analyzeNode(cell)` — record a node
- `analyzeEdge(from, to, RootMarkReason)` / `analyzePropertyNameEdge` / ``analyzePropertyNameEdge`/ `analyzeIndexEdg``analyzePropertyNameEdge`e`analyzeIndexEdg`analyzeIndexEdg``ge``analyzePropertyNameEdge``analyzeIndexEdg``ge`
- `setWrappedObjectForCell(cell, void*)` — link wrapper → native pointer
- `setLabelForCell(cell, String)` — human-readable name in the 快照
- `setOpaqueRootReachabilityReasonForCell` — why a weakly-held wrapper survived

If your 类 shows up as an opaque blob in heap snapshots, 实现 `analyzeHeap`.

## How 迁移到 keep things alive (decision table)

| Scenario                                                                           | Mechanism                                                                                                                                          |
| ---------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| JSCell 字段 pointing 迁移到 another JSCell                                            | `WriteBarrier<T>` member + `visito`visito`.追加(`.追加(m_field)`sitChild`sitChildren`h`vis`visitCh`n`                                              |
| Native state inside the wrapped C++ 对象 holds JS values                         | `visitAdditionalChildren` + register 子空间 as 输出-constraint                                                                                 |
| C++/Zig local across allocation/call                                               | Conservative scan (free) — 添加 `EnsureStillAliveScope` / `值.ensure`value.ensure`tillAlive()`g in`tillAlive()`rs or seeing release-only crashes |
| **Zig** native 对象 holds its own JS wrapper (类 has `finalize: true`)        | **`jsc.J`jsc.J`Re__CODE_`upgrad`d`rad`ork starts, `s, `owngra`down`downgrade()`*This is the default.**                                                  |
| **Zig** native 对象 owns an arbitrary JS 值 (回调, 选项 对象)        | `jsc.Strong.Optional` — `deinit()` `deinit()`z`deinit()`z`deinit()`ze()`finalize()`                                                                |
| C++ non-GC 对象 owns a JS 值 as a root                                        | `JSC::Strong<T>`. **Danger:** cycle if the JS 值 可以 reach back → leak                                                                          |
| Weak ref with resurrection predicate / finalize 回调 (C++)                     | `JSC::Weak<T>` + `Wea`Wea`CODE_2__`                                                                                                             |
| Wrapper kept alive by **many concurrent operations** with no single busy/idle 边 | `.classes.ts` `ha__CODE`PendingActivity: t` t`tomic flag polled on GC 线程). **Uncommon — prefer `r `JSR`r `_CODE_4__SRef`   `                       `you 可以.`
| 分组 of wrappers share lifetime via a native graph                                | `visitor.addOpaqueRoot(ptr)` + `containsOpaqueRoo`containsOpaqueRoo`(ptr)`                    `(ptr)`                                              |
| Temporarily forbid GC in a critical 截面                                        | `DeferGC deferGC(vm)` — defers until scope exit. Never hold across user JS                                                                         |
| Tell GC about off-heap 内存 you own                                              | `reportExtraMemoryAllocated` on alloc **and** `reportExtraMemory`reportExtraMemory`isited``                  `isited`CODE_3__ `         |`                   `
| ~~Mark a 值 as root from C API~~                                                | ~~`gcProtect` / ````JSValueProtect`— **avoid**; use `jsc.St`jsc.St`ong`__CODE_` __CODE`JSRef``                                                                |

## Destruction & finalizers

- `static constexpr bool needsDestruction = true` → C++ destructor runs when the cell is swept. Sweep is **lazy** (next allocation from that block, or `IncrementalSweeper`), so destruction`IncrementalSweeper`ily. Do not rely on it for `IncrementalSweeper`expose explicit `close()`/`dispose()``IncrementalSweeper```close()``dispose()``clos`close()``dispose()``close()``dispose()``close()``dispose()``close()``dispose()`
- In `.classes.ts`, `fi__CODE`alize: t` t`Z__COD`finalize()`nalize()`lize()`rom the destructor. Same laziness applies.
- `WeakHandleOwner::finalize` runs earlier (at weak-reap 时间) but the cell is already dead; only use it 迁移到 清空 caches.
- Destructors 运行 on the mutator 线程 but **other JS objects 可以 already be swept** — do not dereference `WriteBarrier` fields in a destructor.

## Debugging GC issues

```bash
# Force synchronous, frequent GC — turns rare races into immediate crashes
BUN_JSC_collectContinuously=1 BUN_JSC_useConcurrentGC=0 bun-debug test.js

# Zero free cells so UAF reads are obvious
BUN_JSC_scribbleFreeCells=1

# Validate the GC's own bookkeeping
BUN_JSC_verifyGC=1 BUN_JSC_verboseVerifyGC=1

# See what's being collected / heap growth
BUN_JSC_logGC=2 BUN_JSC_showObjectStatistics=1

# Force GC from JS
Bun.gc(true)      // sync full GC
require('bun:jsc').heapStats()
```

If a 缺陷 only reproduces with 并发垃圾回收 **on** → missing 写入 屏障 or `visitChildren` race.
If it only reproduces with `collectContinuously=1` → something isn't rooted across an allocation.
If 内存 grows but `heapStats().heapSize` doesn't → missing `reportExtra`reportExtra`emoryAllocated`emoryAllocated`
If GC runs constantly with little garbage → missing `reportExtraMemoryVisited`.

## 键 source 文件

- `vendor/WebKit/Source/JavaScriptCore/heap/Heap.cpp` — `collectImpl`, `addCoreConstraints` (root`collectImpl``addC`addCoreConstraints``collectImpl``ad`collectImpl`nts`addC``collectImpl``addCoreConstraints`
- `vendor/WebKit/Source/JavaScriptCore/heap/SlotVisitor.cpp` / `SlotVisitorInlines.h` — `drain()`, `追加`, `a`Slo`SlotVisitorInlines.h`aMemoryVisi`dra`drain()`e__CODE__CODE`SlotVisitorInlines.h`o`ExtraMem__TECH_1`dra````e__CODE``_addOpaqueRoo`eportExtraMemoryVisi`oryVisited``append`_CODE_11__`reportExtraMemoryVisited`
- `vendor/WebKit/Source/JavaScriptCore/heap/MarkedBlock.h`, `vendor/WebKit/Source/JavaScriptCore/heap/Prec__CODE_1__seAllocation.h``isLive``isLive``seAllocation.h``isLive``isLive`
- `vendor/WebKit/Source/JavaScriptCore/heap/CellState.h`, `runtime/WriteBarrier.h`, `runtime/Writruntime/WriteBarr__CODE_1__teBa`runtime/WriteBarr``runruntime/WriteBa__CODE_2__h``runtime/WriteBa`__COruntime/WriteBarrierInlines.h
- `vendor/WebKit/Source/JavaScriptCore/heap/ConservativeRoots.cpp`, `vendor/WebKit/Source/JavaScriptCore/heap/MachineStack__CODE_1__arker.cpp``arker.cpp`
- `vendor/WebKit/Source/JavaScriptCore/heap/Weak.h`, `vendor/WebKit/Source/JavaScriptCore/he__CODE_1__p/WeakImpl.h`heap/WeakBlock.h`, `vendorWebKitt/Source/Java`vendheap/WeakBlock.heap/WeakBlvendor/WebKit/Source/JavaWebKitebKit/S__Cvendor/We__CODE_2__heap/WeakBlock.htC`/WeakBlock`kSet.hScriptCorecriptCore/h__COvendor/WebKit/S__CODE_4__ndor/WebKit/Source/JavaScriptC__CODE_5__kSet.hvaScriptCore/heap/WeakHandleO`WeakHandleOwner.h``vendor/WebKit/Source/JavaScriptCore/heap/WeakSet.h``vendor/WebKit/Source/JavaScriptCore/heap/WeakHandleOwner.h`ScriptCore/h__CODE_6__d__CODE_7__vaScriptCore/heap/WeakSet.hvendor/WebKit__CODE_8__JavaScriptCore/heap/WeakHandleO__CODE_9____CODE_10____CODE_11__
- `vendor/WebKit/Source/JavaScriptCore/heap/HeapAnalyzer.h`, `vendor/WebKit/Source/JavaScriptCore/heap/HeapS__CODE_1__apshotBuilder.cpp`p/BunV8HeapSnapshotBuilder.cpp``vendor/WebKit/Source/JavaScriptp/BunV8HeapSnapshotBuilder.cppotBuilder.cpp``vendor/vendor/WebKit/Source/JavaScriptCor__CODE_2__p/BunV8HeapSnapshotBuilder.cppvendor/W__CODE_3__der.cpp
- `vendor/WebKit/Source/JavaScriptCore/heap/DeferGC.h`, `vendor/WebKit/Source/JavaScriptCore/heap/__CODE_1__trong.h`p/HandleSet.h``vendorWebKitt/Source/JavaScriptCop/HandleSet.heSet.vendor/WebKit/Source/JavaScriptCore/heap/HandleSet5__
- `runtime/JSCell.h` / `JSCellI`JSCellI`lines.`lines.h`r lay`r layout, ```visitCh`visitChi`ldren`
- Bun: `src/bun.js/bindings/BunGCOutputConstraint.cpp`, `ZigGeneratedClasses.cpp` (codegen'd `ZigGeneratedClasses.cpp`utputConstraints`)`visitChi`ZigGeneratedClasses.cpp`ints``)``visitOutputConstraints`)``ZigGeneratedClasses.cpp``)``visitOutputConstraints`