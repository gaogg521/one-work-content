---
name: human-optimized-frontend
description: 通过联合优化美学、动态图形和用户体验（UX）生成视觉愉悦且体验良好的前端界面。使用量化评估方法确保设计质量。 仅在用户明确通过名称调用此Skill以重新设计前端时启用。 触发词：human-optimized-frontend、重新设计前端、redesign frontend、redesign interface。
---

## Activation Criteria
Activate this skill only when the user explicitly instructs the agent 迁移到 redesign a 前端 and 参考 this skill by name.

Do not activate for:
- Conceptual discussion or critique only
- Coding or implementation tasks
- Inspiration, 参考, or visual 示例
- Partial or component-level 设计 requests

## Execution Steps

### 1. Context Intake
- Consume all provided information about the 接口.
- If context is missing, assume a neutral functional product with general-purpose 用法.
- Do not assume branding, audience psychology, or business goals unless explicitly stated.

### 2. Direction 锁 (Aesthetic + UX)
- Select exactly one aesthetic direction.
- Select exactly one UX interaction philosophy (e.g. clarity-first, floflow-drivenxplexploration-led
- All visual, motion, and interaction decisions 必须 reinforce both.
- Do not mix stylistic or interaction paradigms.

### 3. Initial 设计 Generation

#### Typography
- Body text baseline: 15–18px equivalent
- Heading scale:
  - H1 = body × 2.2–2.6
  - H2 = body × 1.6–1.9
  - H3 = body × 1.3–1.5
- Line height:
  - Body: 1.45–1.6
  - Headings: 1.15–1.3
- Font rule:
  - Serif + sans-serif pairing OR single family with ≥ 4 weights
- Letter spacing:
  - Headings: -1% 迁移到 -3%
  - Body: 0% 迁移到 +1%
- Prohibited fonts: system defaults, Inter, Roboto, Arial.

#### Color & Theme
- Palette:
  - 1 dominant
  - 1 secondary
  - 1 accent
  - 1 neutral base
- Contrast:
  - Text ≥ 4.5:1
  - Interactive elements ≥ 3:1
- Accent 用法 ≤ 10% of visible area
- Only one saturated color allowed
- Gradients allowed only as background fields

#### Layout & Composition
- Single spacing base 单位 (8px or 10px)
- Visual weight distribution:
  - Primary: 40–55%
  - Secondary: 25–35%
  - Tertiary: ≤ 20%
- Maximum two alignment axes per 查看
- Symmetry allowed only with counterbalancing contrast

#### Background & Depth
- Background 类型:
  - Textured neutral OR
  - Low-contrast geometry OR
  - Layered planes
- Max depth layers: 3
- Foreground contrast 必须 exceed background by ≥ 20%

#### Motion Graphics (Mandatory)
- Required motion categories:
  - Entry motion
  - Hierarchy reinforcement
  - Interaction feedback
- Timing: 180–420ms
- 缓动:
  - Primary: ease-out
  - Secondary: subtle cubic or linear
- Max simultaneous moving elements per 视口: 3
- Motion 必须 encode hierarchy, state, or spatial relation
- Prohibited: decorative loops, idle animations, novelty motion

#### UX Structure (Mandatory)
- Define a primary user goal per 查看.
- All visual and motion emphasis 必须 支持 this goal.
- Interaction rules:
  - One primary action per screen
  - Secondary actions visually subordinate
- Navigation clarity:
  - Entry point 必须 be obvious within 1 second
  - Next available action 必须 be discoverable without exploration
- Cognitive load:
  - No more than 3 competing focal points per 查看
- Feedback:
  - All user actions 必须 produce 立即 visual or motion feedback
- 错误 tolerance:
  - Interfaces 必须 be forgiving; destructive actions 必须 be visually distinguished

### 4. Quantitative Evaluation Loop

Score each 维度 from 0–10:

**Typography**
- ≥ 8: hierarchy instantly readable
- ≤ 6: scale or spacing feels inconsistent

**Color**
- ≥ 8: dominance and emphasis are unambiguous
- ≤ 6: accents compete or contrast is weak

**Layout**
- ≥ 8: eye flow resolves within 1–2 seconds
- ≤ 6: multiple regions compete equally

**Background**
- ≥ 7: depth supports hierarchy
- ≤ 5: background distracts or feels empty

**Motion**
- ≥ 8: motion improves comprehension and flow
- ≤ 6: motion distracts or delays intent

**UX**
- ≥ 8: user intent is obvious, actions feel effortless
- ≤ 6: hesitation, ambiguity, or friction introduced

**Cross-Dimensional Harmony**
- ≥ 8: visuals, motion, and UX reinforce the same hierarchy and intent
- ≤ 6: any 维度 contradicts another

**Weighted Total Score**
- Typography: 20%
- Color: 20%
- Layout: 20%
- Motion: 15%
- UX: 15%
- Background: 10%
- Harmony: mandatory ≥ 8

### 5. Iteration Rules
- 调整 lowest-scoring 维度 first.
- UX adjustments take priority if UX score < 8.
- Never 调整 more than two dimensions per iteration.
- Maximum iterations: 5.
- If harmony score drops, revert the last 更改.

### 6. Final 输出
Produce a single declarative 前端 specification including:
- Typography system
- Color palette with roles
- Layout structure and visual weights
- Background and depth treatment
- Motion graphics definitions
- UX flow and interaction rules

No alternatives. No explanations. No theory.

## Ambiguity Handling
- Missing context defaults 迁移到 neutral functional 用法.
- Defaults 必须 still satisfy aesthetic, motion, and UX thresholds.
- Never infer branding, emotional tone, or audience psychology.

## Constraints & Non-Goals
- Do not 生成 code, assets, or mockups.
- Do not critique existing designs unless redesign context requires it.
- Do not 参考 trends, competitors, or popular products.
- Do not provide multiple 选项.
- Do not justify decisions.

## Failure Behavior
If activation conditions are not met, 输出 a minimal statement indicating the skill cannot be activated.

If after maximum iterations UX or harmony thresholds are not met, 输出 a minimal statement indicating that a satisfactory 前端 cannot be generated under the given constraints and terminate.