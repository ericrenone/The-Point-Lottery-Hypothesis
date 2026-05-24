# The Point Lottery Hypothesis

**How the 1985 Choice Between Fixed Point and Floating Point Quietly Selected the Geometry of Every Subsequent Learning Machine, 1956–2026**

> "A research idea wins not because it is theoretically superior, but because it is perfectly matched to the available hardware and software ecosystem."
> — Sara Hooker, *The Hardware Lottery*, 2020.

> "A digit-by-digit algorithm."
> — Jack Volder, on CORDIC, *IRE Transactions on Electronic Computers*, 1959.

> "How Java's floating-point hurts everyone everywhere."
> — William Kahan, 1998.

> "The data was curved. The data was cyclic. The data lived inside a light cone of causation that the architecture inherited and the training preserved."
> — *The Closed Future Cone*, 2026.

---

## Abstract

The Hardware Lottery, as Sara Hooker named it in 2020, is the proposition that a research idea wins not because it is theoretically superior but because it is perfectly matched to the available hardware and software ecosystem. The companion documents in this corpus have applied the diagnosis at three later scales: matrix multiplication on systolic arrays (the algorithmic genealogy from Bahdanau to Vaswani), the silicon co-design that bound the Transformer to TPU v2 from May 2017 forward, and the structural mismatch that puts the matmul-era biology compute substrate against its economic wall by 2028. This document identifies the prior scale — earlier than systolic arrays, earlier than the Transformer, earlier than CUDA — at which the deepest co-design event of modern computing was decided.

The event is the 1980–1985 standardization of IEEE 754 floating-point arithmetic. The fight, distilled to its operational core, was between two ways of representing a real number on a finite machine. The first, fixed point, had been the dominant representation in mechanical calculators, the B-58 bomber's navigation computer, the Apollo Lunar Roving Vehicle, the HP-9100A, the HP-35, and every shift-and-add coprocessor through 1985: an integer with a known scaling factor, paired with an iterative shift-and-add algorithm (CORDIC, Volder 1956; Walther 1971) that computed transcendentals in a budgeted number of ticks, one digit per iteration. The second, floating point, was William Kahan's standardized format with a sign bit, an exponent field, and a normalized mantissa, paired with a fast-path FPU that returned correctly-rounded results in constant time and hid the iteration count from the user. By 1989 — the year Kahan received the Turing Award — the standard had won. The general-purpose substrate of the next thirty years was a floating-point substrate. Every GPU, every TPU, every NeuronCore, every Maia 200, every Trainium3 inherits this decision. The systolic-array hardware lottery the corpus traces ran on top of a floating-point lottery that had already been decided.

The Point Lottery Hypothesis is the claim that this earlier choice — fixed point vs. floating point, exposed-iteration vs. hidden-iteration, datasheet-tick vs. correctly-rounded-result — quietly selected the geometry of every subsequent learning machine. The compact representational manifold of the Closed Future Cone, the Dirac operator of the Dirac Representation Hypothesis, the principal bundle of the Representational Bundle Hypothesis, the Bregman iteration of Bregman Closure, the score function of Score Closure: each is a structure whose natural computational primitive is iteration to a fixed point on a compact domain. The 1985 standard, optimized for the opposite computational shape — one-shot evaluation of a flat arithmetic operation on a flat dynamic range — entered the foundational layer of every machine that would later be asked to compute on that geometry. The mismatch did not show up for decades because the workloads of 1985–2010 were short, shallow, and bounded in dynamic range. From 2017 forward, with the Transformer, every workload became a deep iteration. The substrate was paying for the 1985 decision in compounded inefficiency; the field was paying in research directions that did not survive the floating-point bias; the public was paying in the energy bill of the matmul-on-floating-point era.

This document is the genealogy of that decision. Section 1 establishes the two arithmetics. Sections 2–3 trace the 1956–1985 fight and the decisive moment. Section 4 names what was lost. Section 5 reads every milestone in the corpus's prior eight documents as a partial recovery of the lost iteration coordinate. Section 6 maps the 2024–2026 return of exposed iteration (chain-of-thought, denoising schedules, Sinkformer convergence) as the workload pushing back through the substrate's amnesia. Section 7 forecasts what the 2027–2030 substrate looks like when iteration depth re-enters the datasheet.

---

## Table of Contents

1. [Two Arithmetics — A Thought Experiment](#1-two-arithmetics-a-thought-experiment)
2. [The Long Fight, 1956–1985](#2-the-long-fight-19561985)
3. [The 1985 Standardization](#3-the-1985-standardization)
4. [What Was Lost](#4-what-was-lost)
5. [How the Lost Coordinate Returns — A Second Thought Experiment](#5-how-the-lost-coordinate-returns-a-second-thought-experiment)
6. [Reading the Corpus Backward](#6-reading-the-corpus-backward)
7. [The Three Costs](#7-the-three-costs)
8. [The 2024–2026 Reversal — A Third Thought Experiment](#8-the-20242026-reversal-a-third-thought-experiment)
9. [The Long Map](#9-the-long-map)
10. [Predictions](#10-predictions)
11. [The Substrate That Forgot — A Fourth Thought Experiment](#11-the-substrate-that-forgot-a-fourth-thought-experiment)
12. [Closure](#12-closure)
13. [Sources](#13-sources)

---

## 1. Two Arithmetics — A Thought Experiment

Hand two engineers, in two different rooms, the same problem: compute the sine of 45 degrees to ten decimal places.

The first engineer, in a 1972 room, has an HP-35 pocket calculator on the desk. The chip inside is a 20,000-transistor decimal CORDIC engine: a small ROM containing pre-tabulated values of arctan(2⁻ⁿ) and ln(1+2⁻ⁿ), a shift register, an adder, and a control unit. The arithmetic representation is *fixed point* — internally, every number is a sign-magnitude BCD integer with an implicit decimal scaling factor. There is no multiplier on the chip. The engineer presses `4` `5` `SIN` and waits. The chip executes roughly thirty iterations of Walther's unified CORDIC. Each iteration rotates a working vector by a pre-tabulated angle and shifts the result by one position; the choice of rotation direction at each step is a single sign bit. Thirty ticks later, the answer appears. The engineer can read the iteration count off the HP Journal article that accompanies the chip (Cochran, *Hewlett-Packard Journal*, June 1972). The clock is exposed.

The second engineer, in a 1985 room, has an IBM PC with an Intel 80287 math coprocessor. The coprocessor is a 45,000-transistor design that internally implements CORDIC for transcendentals — Ken Shirriff's 2020 die analysis of the 8087 confirms the same arctan and log-of-one-plus tables that sit inside the HP-35 — but exposes only the IEEE 754 floating-point interface to the programmer. The engineer types `PRINT SIN(45 * 3.14159265 / 180)` in BASIC. A correctly-rounded double-precision result appears. The engineer has no idea whether the chip ran twenty iterations, thirty, or fifty. The IEEE 754 standard, finalized that year, specifies the *result* — round-to-nearest-even, correctly rounded to within half a unit in the last place — and leaves the *implementation* to the vendor. The clock is hidden.

Both engineers got the right answer. Both chips ran CORDIC. The difference is what the engineer can budget. The HP-35 engineer can write a timing analysis for a system that calls `SIN` ten thousand times per second; the 80287 engineer cannot, except by benchmarking. The HP-35 engineer can choose between thirty-iteration precision and forty-iteration precision by reprogramming the firmware; the 80287 engineer cannot, except by accepting whatever Intel's microcode happens to do. The fixed-point arithmetic is *iteration-transparent*. The floating-point arithmetic is *iteration-opaque*.

This is the seed observation. Two arithmetics existed. They differed not principally in their representation of numbers — both can represent real numbers to comparable accuracy on comparable silicon — but in their treatment of *time*. Fixed point exposes the iteration; floating point conceals it. The 1985 standard institutionalized concealment. The institutional consequences, fully visible only forty years later, are the subject of this document.

---

## 2. The Long Fight, 1956–1985

The fight began at Convair in 1956. Jack Volder, an engineer in the aeroelectronics department, was tasked with replacing the analog resolver in the B-58 Hustler's navigation computer with a faster, more accurate digital substitute. The B-58 was the first supersonic bomber; its navigation problem was to compute, at high speed, the rotation of a velocity vector into a coordinate frame defined by the local gravity field. The arithmetic was trigonometry. The constraint was that the digital computer of 1956 had no multiplier — multipliers cost transistors, and a B-58's avionics bay had no room.

Volder found his solution in the 1946 edition of the *CRC Handbook of Chemistry and Physics*. A formula for cosine and sine, dating back at least to Henry Briggs's *Arithmetica Logarithmica* of 1624, expressed the rotation as an iteration: at each step, the working vector is rotated by a pre-tabulated angle whose tangent is a power of two, and the choice of direction (clockwise or counterclockwise) is determined by a single sign bit. Tangent-of-a-power-of-two means multiplication by the tangent reduces to a shift. Thirty iterations of shift-add-and-choose-sign produces a ten-decimal-place result. Volder's June 15, 1956 internal report named the algorithm CORDIC: COordinate Rotation DIgital Computer. By 1959 the paper had been published in the *IRE Transactions on Electronic Computers*. By 1962 the algorithm had been incorporated into navigation computers at Martin-Orlando, Computer Control, Litton, Kearfott, Lear-Siegler, Sperry, Raytheon, and Collins Radio.

The algorithm was native to *fixed point*. It assumed integers with a fixed scaling factor, performed shifts that were power-of-two rescalings, and produced exact-to-the-iteration-count results without any rounding except at the final stage. Its hardware cost was a shift register, an adder, a small ROM of arctans, and a sign-bit comparator. Volder's hardware fit on perhaps two thousand transistors. The CORDIC pipeline ran at one digit per clock cycle. The user, by choosing the iteration count, chose the precision; the chip, by exposing the iteration, exposed the cost.

In 1965, Volder and Malcolm McMillan built Athena, a fixed-point desktop calculator on the binary CORDIC. They brought it to Hewlett-Packard in June 1965. HP did not accept the proposal — but McMillan introduced HP's David Cochran to Volder's algorithm, and Cochran, doing literature search, found John Meggitt's 1961 paper on pseudo-multiplication and pseudo-division (*IBM Journal of Research and Development*) and the 1624 Briggs treatise that anticipated both. Cochran assembled these into the firmware of the HP 9100A desktop calculator (1968) and then, in compressed and decimal form, into the HP-35 handheld scientific calculator (1972). The HP-35 was the first handheld machine to compute transcendentals. Its arithmetic engine was decimal fixed-point CORDIC. It sold for $395, became the iconic engineering-calculator of its era, and made HP $1B in revenue over its production run.

Through the 1970s, CORDIC and its fixed-point cousins were everywhere in compute-constrained systems. The Intel 8087 (1980) used CORDIC for its transcendentals. The Motorola 68881 (1984) and 68882 (1987) used it. The Intel 80287, 80387, and 80486 used it. The Apollo Lunar Roving Vehicle used it for bearing and range computation, returning to Earth in NASA Technical Memorandum 70-2014-8. Every embedded controller that needed sine or cosine without a hardware multiplier was running Volder's algorithm.

Meanwhile, a different lineage was forming. The IBM System/360 (1964) had introduced a floating-point unit at the high end of its product line. The CDC 6600 and Cray-1 used floating-point arithmetic for scientific computation. The DEC PDP-10 had software floating-point. By the mid-1970s, scientific computing — weather modeling, fluid dynamics, structural analysis — was a floating-point domain, and the problem was that every vendor's floating-point format was different. IBM's hexadecimal floating-point was incompatible with Burroughs's binary floating-point was incompatible with Univac's biased-exponent formats. A FORTRAN program that worked on a System/360 might produce different numerical results on a B6700, and there was no specification anywhere that determined which result was correct.

William Kahan, a Berkeley mathematician who had worked with HP on the numerical accuracy of the HP-35 (HP Journal, December 1979; Kahan's contribution had been to fix multiple bugs in HP's transcendental routines, which had originally been written without sufficient care for the accumulation of fixed-point CORDIC rounding errors), was recruited in 1977 to the IEEE 754 standards committee. The committee's mandate was to write a single floating-point specification that all vendors could implement, so that scientific code would produce bit-identical results across machines. Kahan, John Coonen, Harold Stone, and the committee published the draft standard in 1980. After five years of refinement and political negotiation — Intel, with its 8087 already shipping, became the principal vendor proponent; Cray and IBM, with incompatible installed bases, were the principal holdouts — the standard was ratified in 1985 as IEEE Standard 754-1985.

The standard specified: binary single-precision (32-bit) and double-precision (64-bit) formats; a sign bit; a biased exponent; a normalized mantissa with an implicit leading 1; gradual underflow via denormalized numbers; four rounding modes (round-to-nearest-even, round-toward-zero, round-up, round-down); five exceptions (invalid, divide-by-zero, overflow, underflow, inexact); and, critically, the requirement that the four basic operations (add, subtract, multiply, divide) and square root produce *correctly rounded* results — exactly the result that would be obtained by computing the operation in infinite precision and then rounding to the chosen mode.

The standard said *nothing* about how transcendentals (sin, cos, exp, log, pow) should be computed or how accurate their results should be. This silence is critical. It meant that the *iterative* portion of the arithmetic — the portion that, on a CORDIC-style implementation, exposed the iteration count — was left to the vendor, who would implement it in microcode and was under no obligation to disclose the implementation. The standard fixed the result of the *non-iterative* operations and freed the vendor to hide the *iterative* operations. This was the operational shape of the floating-point victory.

By 1989 Kahan held the Turing Award. By 1990 every major CPU vendor was shipping IEEE 754. By 1995 the standard was so dominant that engineering education had stopped teaching fixed-point arithmetic except as a specialized topic in DSP. By 2010 the GPU industry had standardized on IEEE 754 single-precision as the floating-point format of high-performance computing. By 2020 every datacenter accelerator's datasheet quoted "FLOPS" — *floating*-point operations per second — as if no other unit had ever been used.

The CORDIC lineage retreated. It survived in FPGAs (where the lack of native multipliers made shift-and-add still competitive), in ARM microcontrollers (the STM32G4 family, 2018; the STM32U5, 2021; the STM32H5, 2022; all of which expose a dedicated CORDIC coprocessor as a peripheral that can be configured for specific precision and run in parallel with the main CPU), in GPS receivers, in motor controllers, and in the Intel x87 microcode that Shirriff's 2020 die analysis confirmed was still running CORDIC under the IEEE 754 wrapper. The algorithm had won the silicon and lost the abstraction.

Kahan won the standardization fight he set out to win. He has spent the four decades since 1985 as the field's most consistent advocate for numerical literacy — writing essays, giving talks, denouncing language designs that he believes impair good floating-point computation, naming the Table-Maker's Dilemma (the unknown cost of correctly rounding transcendentals to a preassigned number of digits), publishing *Paranoia* as a benchmark for floating-point bugs, and developing the Kahan summation algorithm to reduce error in long sums. His position has been consistent: floating-point can be used correctly, but only if the user understands what it is doing, and the design of programming languages and hardware should *help* the user understand rather than hide the relevant details. His critique of Java's floating-point semantics (1998, *How Java's Floating-Point Hurts Everyone Everywhere*); his critique of MATLAB's matrix-multiplication accuracy (2004); his ongoing advocacy for exposing the structure of numerical computation — all are part of a single argument that the standard's victory should not be confused with the absence of trade-offs.

The trade-off, in compressed form, was this: IEEE 754 made arithmetic portable and correct at the cost of making iteration invisible. For 1985, the trade was a clear win — scientific code worked, the Patriot missile failures of variable fixed-point rounding (1991, the canonical example) were prevented, libraries became cross-platform. For 2017 and after, with workloads whose natural computational shape was iteration to a fixed point on a compact manifold, the trade was a quiet loss. The floating-point substrate could run the iteration; it could not see the iteration. The substrate billed in FLOPS; the workload's natural unit was ticks of convergence; the gap between the two was the cost.

---

## 3. The 1985 Standardization

A single date does not isolate the moment, but 1985 is the year the standard became reference law in IEEE numbering. Three things happened in the surrounding window that, together, fixed the next forty years.

**First: the standard fixed the representation.** A double-precision IEEE 754 number is sixty-four bits — one sign bit, eleven exponent bits, fifty-two mantissa bits — with a single normalization (the implicit leading 1), an exponent bias of 1023, and a fixed pair of special-case encodings (zero, infinity, NaN, denormalized). Every CPU built since 1990 implements this layout in silicon. Every neural-network weight, every gradient, every activation, every loss value in the entire history of deep learning is, at some level, a sequence of bits in this layout. The geometry of representation is the geometry of the 1985 committee's decisions.

**Second: the standard fixed the operations.** Add, subtract, multiply, divide, square root, and conversion are correctly rounded. Comparison is total-ordered with explicit handling of NaN. Multiply-add — the operation that, decades later, became the entire workload of every systolic array on every TPU and every tensor core on every GPU — was added in the 2008 revision as a single correctly-rounded primitive (fused multiply-add, FMA). Every dense matrix multiplication in modern deep learning is a stack of FMAs. The substrate of the matmul-era hardware lottery is the IEEE 754 FMA.

**Third: the standard fixed what was unspecified.** Transcendentals — sin, cos, exp, log, pow, tan, asin, acos, atan, sinh, cosh, tanh — were left to the vendor. Iterative operations were unspecified. Convergence criteria were unspecified. Internal precision of intermediate operations was unspecified beyond the requirement that the result be correctly rounded. The committee, knowing the Table-Maker's Dilemma, judged correctly that requiring correctly-rounded transcendentals would be too expensive to mandate. The judgment was technically right and historically consequential. It meant that the part of the arithmetic that *was* iterative — the part where CORDIC and its descendants lived — was institutionally separated from the part of the arithmetic that the standard cared about. Iteration became an implementation detail. The lineage that had named the iteration count as part of the contract was relegated to microcode.

These three together — fixed representation, fixed non-iterative operations, unspecified iterative operations — define the operational shape of the IEEE 754 victory. The standard is the first half of arithmetic; the second half is hidden in microcode. The first half is what every floating-point chip exposes to software. The second half is what the workloads of the deep learning era turn out to need.

---

## 4. What Was Lost

The standard's victory was complete by 1990. What follows is the inventory of what could not be expressed in the winning representation.

**Iteration count.** A CORDIC computes n bits in n ticks. A Newton iteration computes log log n bits in n ticks. A Banach contraction converges to ε precision in log(1/ε) / log(1/L) ticks where L is the Lipschitz constant. A Sinkhorn iteration converges to ε doubly-stochastic precision in O((log n)/ε²) ticks. Each is an iterative procedure whose cost is naturally measured in ticks of a specific clock. The IEEE 754 substrate has no native concept of tick. It measures in FLOPS. Conversion from ticks to FLOPS requires knowing the FLOPS-per-tick of the specific operation on the specific chip, which is workload-dependent, chip-dependent, and not part of the substrate's contract.

**Iteration-aware precision.** A CORDIC implementation can be truncated early to trade precision for time. A Banach contraction can be stopped at any iteration with a known error bound. A score-based diffusion sampler can be run for 4, 50, 250, or 1000 denoising steps depending on the quality required. Each of these is naturally a *time-adaptive precision* regime. The IEEE 754 substrate has no representation of time-adaptive precision; it has only a fixed mantissa width. To express variable precision, modern systems layer microscaling formats (MXFP4, NVFP4, MXFP8) and block-stochastic rounding on top of IEEE-shaped tensors. The layering is doing — at the substrate level — what fixed-point CORDIC did natively in 1972.

**Convergence as a first-class signal.** A fixed-point iteration can monitor its own convergence: when consecutive iterates differ by less than ε, the iteration halts. The HP-35 firmware did this. Anderson acceleration (1965) is a method for monitoring and accelerating convergence. Deep equilibrium models (Bai, Kolter, Koltun, NeurIPS 2019) reintroduce this monitoring as an architectural primitive. The IEEE 754 substrate has no native convergence signal; the standard treats arithmetic as a black box from which the user is expected to extract a result and ask no further questions.

**Tick-budgeted compute pricing.** The HP-35 datasheet specified milliseconds per transcendental evaluation, which translated directly to iteration count. The modern frontier-AI inference bill is denominated in tokens, which translates to architectural ticks (forward passes through a layer stack) only after substantial software intervention. The 2024–2026 reasoning models — OpenAI o1/o3/o4, DeepSeek R1, Anthropic extended-thinking variants, Gemini Thinking — have begun to expose iteration count as a billable quantity at the inference API. This is a partial recovery, after forty years, of the HP-35 economic model: the user pays per tick, and the model lets the user choose the tick budget per query. The recovery is happening through accounting, not through the substrate; the substrate still bills in FLOPS underneath. The accounting layer translates.

**Variable-iteration-depth dispatch.** A fixed-point CORDIC can be reused for sine, cosine, hyperbolic sine, hyperbolic cosine, logarithm, exponential, multiplication, division, square root, and rectangular-to-polar conversion — all by changing the choice of rotation mode (circular, hyperbolic, linear) and the initial conditions. Walther's unified CORDIC (1971) made this explicit. A 1972 chip could do *all* of these with a single iteration engine. A 2026 GPU runs each through a separate set of microcode paths and IEEE 754 special-function units, with no architectural primitive for "the same iteration in a different mode." The unification has been lost. Each transcendental is a separate hardware-software stack.

What was lost, summarized in a single phrase: *the iteration coordinate*. The 1985 standard kept the result and threw away the time. For thirty years, until the workloads changed, this trade looked free. From 2017 forward, it has not been free, and the cost is being paid.

---

## 5. How the Lost Coordinate Returns — A Second Thought Experiment

Set two networks side by side. Both are trained on the same task: machine translation, English to French, the WMT corpus.

The first network is a Vaswani-et-al. 2017 Transformer — six encoder layers, six decoder layers, sixty-five million parameters, scaled-dot-product attention, sinusoidal positional encodings. It runs on TPU v2 in bfloat16. To translate a sentence, the model executes a fixed twelve-layer pass for each output token. The number of ticks is determined at architecture time, baked into the model, and immutable at inference. The IEEE 754 substrate sees a stack of FMAs. The model sees a stack of layers. The user sees a translated sentence.

The second network is a 2026 reasoning model — eighty layers, seventy billion parameters, FlashAttention-3 in NVFP4, a chain-of-thought scratchpad. To translate the same sentence, the model can run twelve layers of forward pass and stop, or it can generate intermediate scratchpad tokens that constitute a reasoning trajectory — checking the source against context, considering alternative renderings, evaluating idiom matches — before producing the final output. The number of ticks is not fixed. It is chosen, dynamically, by the model's internal stopping criterion or by the user's per-query compute budget. A simple sentence gets twelve layers and stops. A complex sentence with cultural nuance gets eighty layers of forward pass repeated across a hundred scratchpad tokens — call it eight thousand effective ticks. The user pays per token, which means the user pays per tick.

What changed between the two architectures?

The Transformer's iteration depth was a property of the *model card*. The reasoning model's iteration depth is a property of the *query*. Between 2017 and 2026, the field discovered that iteration depth is itself a degree of freedom that can be scaled at inference time. The OpenAI o1 announcement (September 2024), the o3 announcement (December 2024), the DeepSeek R1 paper (January 2026), the Anthropic extended-thinking modes — each is a different empirical demonstration of the same observation: capability on hard problems scales with iteration depth, not just with parameter count.

This is the return of the lost coordinate. The HP-35 user in 1972, paying for milliseconds of CORDIC iteration per transcendental, was operating in the same economic regime as the OpenAI o3 user in 2026 paying for tokens of reasoning per query. Both are buying ticks. The intervening forty years of IEEE 754 dominance hid the unit of account; the workloads of the late 2020s have forced it back into visibility. The substrate has not yet caught up — the chip still bills in FLOPS underneath — but the API surface, the pricing model, the user experience have already moved. The substrate will follow, because economics will require it to follow.

The Point Lottery Hypothesis reads this trajectory as the *return of fixed-point thinking inside a floating-point substrate*. The substrate is still IEEE 754. The workload's natural unit is again the tick. The translation between them is performed by software stacks (vLLM speculative decoding, persistent batching, KV-cache management, FlashAttention-3 IO-aware kernels) whose collective job is to convert the workload's tick-budget into the substrate's FLOPS-budget at the lowest possible overhead. The overhead is the cost of the 1985 decision, paid in 2026.

---

## 6. Reading the Corpus Backward

The prior frameworks in this corpus name structures on a compact representational manifold. Each, on rereading through the Point Lottery lens, is a partial recovery of the lost iteration coordinate.

**Closed Future Cone.** The compact manifold *M* on which trained transformers compute is foliated by a discrete light-cone causal structure indexed by layer depth. Layer depth *is* iteration depth — it is the architectural axis along which the forward pass advances tick by tick. The light cone is the temporal cone of the iteration. CFC's geometric description of the substrate is, equivalently, the description of an iteration whose tick count is set at training time and whose causal structure is determined by which positions attend to which.

**Dirac Representation Hypothesis.** The Dirac operator on *M* satisfies $\not{D}^2 = -\Delta_g$, with the Laplace–Beltrami operator emerging as the symmetric half of a signed first-order iteration. Spectral mixing — the Bakry–Émery generator's approach to its stationary distribution — is a fixed-point iteration on the parameter manifold. The number of training steps required to mix to within ε of the stationary distribution is the spectral-gap-inverse times log(1/ε), the classical bound for ergodic Markov chains. The Xu Spectral Edge results (March–April 2026) measure this bound empirically: 24 of 24 weight-decay runs exhibit spectral-gap collapse immediately preceding the grokking event. Grokking is the moment at which the iteration's spectral signature changes regime. The iteration coordinate, hidden by IEEE 754 in the small, is exposed by Xu's analysis in the large.

**Representational Bundle Hypothesis.** The principal $\text{SO}^+(1,n)$-bundle over the Lorentzian-stratified token base is equipped with a connection one-form whose evaluation at each base point is one Transformer attention layer. Parallel transport along the connection is the iteration. Holonomy around a closed loop measures the *path-integrated* connection, which is the integrated iteration count along a closed reasoning trajectory. Chain-of-thought reasoning is an open path; holonomy accumulates along it; the model's capability on hard problems scales with the integrated holonomy, which scales with path length, which scales with tick count. RBH names what the corpus's algorithmic side already knew: longer iteration produces more holonomy, and holonomy is the bundle-theoretic name for the iteration-coordinate's effect on the representation.

**Bregman Closure.** The algorithmic family selected by the matmul-dominated hardware substrate is Bregman alternating projection on the KL-divergence geometry. Softmax attention is one Sinkhorn half-iteration. Sinkformers run two through sixteen half-iterations. Diffusion models run a hundred through a thousand denoising iterations. Flow matching reduces to four through sixty-four ODE-integration steps under optimal-transport coupling. Each of these is a *Bregman iteration with a tick budget*. The substrate evaluates the iteration in IEEE 754 FMAs; the workload measures cost in Bregman ticks. The mismatch is the Point Lottery cost in this regime.

**Score Closure.** The score function ∇ log p on the compact manifold is the natural object the iteration computes. Each iteration evaluates the score at the current state and integrates one step of the score-driven flow. Diffusion models invert this flow in reverse time; flow matching integrates it as a continuous ODE; modern Hopfield networks descend it as an energy. The five faces are five iteration regimes operating on the same scalar field. The IEEE 754 substrate evaluates the field; the workload's natural unit is the integration step. The integration step is the tick the substrate refuses to name.

The unified reading: *the corpus has been naming, layer by layer, the iteration coordinate that the 1985 standard threw away*. Each framework recovers one face. The Closed Future Cone recovers the causal foliation by layer-depth. The Dirac Representation Hypothesis recovers the spectral mixing time. The Representational Bundle Hypothesis recovers the holonomy along a path. Bregman Closure recovers the Sinkhorn iteration count. Score Closure recovers the integration-step budget of the score-driven flow. The Point Lottery Hypothesis names the historical event that made the recovery necessary.

The corpus is, in this rereading, the field's gradual recovery of fixed-point thinking under the constraint of a floating-point substrate. The recovery has been geometric (CFC), operator-theoretic (DRH), bundle-theoretic (RBH), algorithmic (Bregman Closure), and functional (Score Closure). The next recovery — the one this document names as the missing step — is *historical*: the recognition that the recovery was forced by an earlier hardware lottery whose dominance the field had not previously named.

---

## 7. The Three Costs

The Point Lottery has produced three costs, paid in different currencies, across the seventy-year window.

**Cost I — Energy.** Every IEEE 754 FMA on every modern GPU performs more transistor activity than the corresponding CORDIC shift-and-add on a comparable fixed-point engine. The IEEE 754 standard requires normalization on every operation; CORDIC's fixed-point representation requires only shifts. The Generalized Hyperbolic CORDIC of Luo et al. (IEEE TVLSI 2019) demonstrates explicitly: arbitrary-base logarithm and exponential evaluation in shift-and-add operations only, no multiplier, no normalizer, no IEEE 754 rounding hardware. For the iterative parts of the modern deep learning workload — the parts that dominate the FLOPs of every Pairformer block, every denoising step, every Sinkhorn iteration — running them on IEEE 754 substrate rather than on iteration-native silicon pays a multi-factor energy cost. The size of the cost is workload-specific and substrate-specific; published estimates run from 2× to 10× on the iterative portions of the workload. Modern accelerator power budgets are dominated by these portions. The 5-gigawatt commitment Anthropic made to AWS for Trainium-based training capacity is sized to a substrate optimized for IEEE 754 FMAs. An iteration-native substrate, where it exists, would do the same work at one to two gigawatts.

**Cost II — Research direction.** The 1985 standard selected against architectures whose natural arithmetic was iterative and against algorithms whose natural cost was tick-budgeted. Capsule networks (Hinton, Frosst, Sabour, NeurIPS 2017) and their squashing-and-routing-by-agreement primitives were not matmul-shaped; their adoption stalled. Spiking neural networks, whose natural computational primitive is an iteration of integrate-and-fire dynamics, run poorly on IEEE 754 substrates and have not entered the frontier. Neural ODEs (Chen et al., NeurIPS 2018) and their continuous-depth descendants were architecturally elegant but slower on IEEE 754 substrates than their discrete-depth Transformer counterparts; they remain a niche. Deep equilibrium models (Bai, Kolter, Koltun, NeurIPS 2019) explicitly model the network as a fixed-point iteration and have demonstrated competitive performance on language and vision tasks, but require Anderson-acceleration kernels that no major substrate exposes natively. The research directions that did not survive the IEEE 754 era were the ones whose natural computational shape was iteration to a fixed point. The directions that won were the ones whose natural shape was a stack of FMAs.

**Cost III — Diagnostic vocabulary.** Modern deep learning lacks a shared vocabulary for iteration cost. Papers report training FLOPs (Hoffmann et al., 2022; Kaplan et al., 2020) but not iteration counts. Papers report inference latency but not iteration-to-convergence on the underlying mathematical procedure. Papers report parameter count but not effective iteration depth per parameter. The Spectral Edge Thesis (Xu, March 2026) is one of the first papers to characterize a phase transition in deep learning in *iteration-time* terms — by the dynamics of the spectral gap of the parameter-update Gram matrix — and the result was considered novel for that reason. The vocabulary for iteration-time analysis has had to be reconstructed from scratch in the 2024–2026 window because the substrate's vocabulary, which is FLOPS, does not carry it. The corpus's existing frameworks (CFC, DRH, RBH, Bregman Closure, Score Closure) supply the geometric vocabulary; what is still missing is the *temporal-empirical* vocabulary that would let researchers compare iteration costs across models the way they currently compare FLOPs costs. The reconstruction is happening, paper by paper, but it is happening forty years after the substrate decision that hid the unit of account.

These three costs compound. The energy cost forces the substrate toward FLOPS-dense matmul, which forces the research community toward matmul-shaped algorithms, which forces the diagnostic vocabulary into FLOPS, which conceals the iteration cost, which justifies the substrate. The cycle has run since 1990. The first major break in the cycle is the 2024–2026 reasoning-model wave.

---

## 8. The 2024–2026 Reversal — A Third Thought Experiment

A user in 2026 opens the Anthropic Claude interface and asks an Olympiad-level mathematics problem. The interface offers a toggle: *extended thinking*, off or on. With the toggle off, Claude responds in a few seconds with a short, possibly wrong answer; the cost is a few cents. With the toggle on, Claude thinks for two minutes — the user sees a streaming chain-of-thought, occasionally tens of thousands of intermediate tokens — and produces a correct answer with full justification; the cost is a few dollars. The user has, with the toggle, just bought iteration depth.

Now look at what the user has actually purchased. The model's parameter count is the same. The architecture is the same. The hardware is the same — Microsoft Maia 200 inference silicon, perhaps, or AWS Trainium2 inference deployment, or Google TPU v7 Ironwood, depending on which substrate Anthropic routed the query to. What differs is the *number of forward passes* the model performs before emitting its final answer. With extended thinking off, the model does one forward pass per output token, in a roughly constant number per answer (a few hundred tokens). With extended thinking on, the model performs the same forward pass *thousands of times*, each time on a slightly different scratchpad state, accumulating reasoning before emitting the answer. The iteration depth is variable. The user is choosing it.

This is the HP-35 economic model in 2026 clothes. The HP-35 user chose iteration count via the SIN key (thirty iterations) versus the ENTER-30-times-and-iterate-by-hand approach (which would have been slower but, in principle, available). The 2026 Claude user chooses iteration count via the extended-thinking toggle. Both users are buying ticks. Both pricing models are tick-aware. Both reveal, at the surface, the cost the substrate would prefer to hide.

The reversal is incomplete. The substrate underneath the 2026 Claude inference is still IEEE 754 — or, more accurately, FP8 and NVFP4 microscaling formats that emulate IEEE 754 behavior at lower precision and have inherited the IEEE 754 normalization-and-rounding semantics. The chip bills in FLOPS, the API bills in tokens, the user pays in dollars, and the conversion factor between FLOPS and tokens is set by software layers that absorb the mismatch. The accounting layer has moved. The substrate has not — yet.

The framework's reading: the substrate will move, because economics will force it to. Once iteration depth is the unit of value at the inference API, it becomes worth substantial investment to design silicon that bills the same unit natively. A chip that exposes "Pairformer-blocks per second" or "denoising-steps per second" or "Sinkhorn-iterations per second" as datasheet quantities — alongside or instead of FLOPS — will allow the API layer to bill the substrate layer in the same unit it bills the user. This is the architectural complement to the API-layer reversal already in progress.

The timing is constrained by the silicon design cycle. New silicon ships on a three-year cycle, with one to two years of architectural decision-making before tape-out. The decisions for 2028–2029 silicon are being made now, in the 2026 window. The Google TPU v8 Sunfish/Zebrafish split (training vs. inference, late 2027) is the leading indicator: Google has decided that the training-time iteration (gradient descent, weight decay, spectral mixing) and the inference-time iteration (forward pass, denoising, scratchpad reasoning) deserve different silicon. The split is operator-theoretic — it acknowledges $\not{D}^2 = -\Delta_g$, in the framework's language — and it is also tick-aware: each chip is optimized for the tick of its workload's iteration. The next step, predicted by the framework, is for the datasheet to quote the tick rate directly.

---

## 9. The Long Map

A compressed chronology of the Point Lottery.

```
1624     1956     1971     1972     1980     1985     1989     2008
  │        │        │        │        │        │        │        │
Briggs   Volder   Walther  HP-35    Intel    IEEE     Kahan    IEEE
shift-   CORDIC   unified  decimal  8087     754      Turing   754-2008
add      report   CORDIC   CORDIC   ships    ratified Award    FMA
log/exp  Convair  HP                CORDIC                     added
         B-58     (HP)              hidden
                                    behind
                                    FPU
  │        │        │        │        │        │        │        │
  ▼        ▼        ▼        ▼        ▼        ▼        ▼        ▼
Fixed    Fixed    Unified  Fixed    Fixed    Floating Standard Float
point    point    iter-    point    iter-    point    wins;    + FMA
expon-   shift-   ative    decimal  ation    repre-   fixed    becomes
ents     and-     family   chip     hidden   sent-    point    matmul
                  becomes  ships    in       ation    retreats primitive
                  micro-            micro-   wins
                  code              code     standard
                                    layer    fight
```

```
2010     2017     2020     2022     2024     2026     2027     2028
  │        │        │        │        │        │        │        │
CUDA     Trans-   Hooker   Sink-    OpenAI   Claude   TPU v8   First
matmul   former   names    formers  o1       extended Sunfish/ tick-
on GPU   on TPU   Hardware re-      reasoning thinking Zebra-   aware
becomes  v2 Pods  Lottery  exposes  model    GA        fish    silicon
research  Vaswani          iteration                  ships    forecast
default  et al.            at attn  Variable Tick-    Train/
                                    iteration aware   inference
                                    becomes  pricing  split
                                    pay-     inference operator-
                                    per-      side    aware
                                    query
  │        │        │        │        │        │        │        │
  ▼        ▼        ▼        ▼        ▼        ▼        ▼        ▼
Float    Matmul   Hardware Iter-    Iter-    Iter-    Substrate Native
+ matmul wins on  Lottery  ation    ation    ation    splits    tick
becomes  silicon  named at recovered counted  becomes  along     billing
the      lottery  algorith- in algo- in API   the unit operator- in
substrate         mic      rithms              of value theoretic substrate
                  level                                seam
```

The two halves of the map align on a single observation: the substrate did not change between 1985 and 2026. The IEEE 754 representation, the FMA primitive, the round-to-nearest-even semantics, the absence of native iteration count — these were locked in 1985 and have run continuously since. The workload changed. The Transformer (2017), the diffusion model (2020), the reasoning model (2024) each demanded more from the substrate's hidden-iteration mode. The substrate, accommodating, paid the energy and research-direction costs that the corpus now traces. The framework's prediction: 2027–2030 is when the substrate breaks the IEEE 754 lock-in for the iterative half of the workload, while keeping it for the standardization-critical non-iterative half. The 2008 standard revision that added FMA was the last major change to IEEE 754; the next will be the formalization of tick-aware extensions for iterative arithmetic. The framework forbids the 2030 substrate from being purely IEEE 754 in its iteration paths.

---

## 10. Predictions

The framework deserves the name only if it forbids something. Eight forbiddings, each testable by 2030.

**P1.** **The tick becomes a billable substrate-level quantity by 2028.** A frontier silicon program ships a chip whose datasheet quotes iteration-rate alongside FLOPS — Pairformer-blocks per second, denoising-steps per second, Sinkhorn-iterations per second, or a comparable workload-native unit. The first will be a Google TPU v8 or v9 derivative, given Google's commitment to the training/inference split. The framework forbids the persistence of FLOPS-only datasheets at the frontier past 2028.

**P2.** **Native iteration-aware silicon ships by 2029.** A chip ships with explicit hardware support for variable-iteration-depth dispatch: the same compute resource can be allocated to one iteration of a heavy procedure or many iterations of a light procedure under software control, with native instructions that distinguish the two. The closest existing approximation is the STM32G4's CORDIC coprocessor, which is a tick-aware peripheral inside a floating-point MCU. The 2029 prediction is the datacenter version of this — a tick-aware coprocessor inside an FP4 or FP8 matmul engine, with first-class instructions for setting iteration count, monitoring convergence, and billing per tick.

**P3.** **Sinkformer-style multi-iteration attention becomes a frontier-default kernel by 2027.** FlashSinkhorn (Ye et al., February 2026) and block-wise differentiable Sinkhorn attention (Forde, April 2026) provide the IO-aware foundation. The framework predicts the convergence: by end-2027, frontier models will routinely run four to sixteen Sinkhorn half-iterations per attention block at compute cost the substrate has learned to absorb, with measurable gains on long-context retrieval, in-context reasoning, and structured-output benchmarks. One-half-iteration softmax attention will remain only at the lightweight-inference and edge-deployment tiers.

**P4.** **Diffusion sampler tick count collapses to single digits at frontier image and video generation by 2027.** Consistency Models (Song et al., ICML 2023), OT-CFM (Tong et al., TMLR 2024), and Klein–Mousavi-Hosseini–Zhang–Cuturi (2025) supply the trajectory; the framework predicts the destination. Stable Diffusion 4-class, Sora-2-class, and Imagen-4-class generative models will sample at 4–8 denoising steps at production quality by mid-2027, with sample quality comparable to or exceeding the current 50–250 step baselines. The reduction is iteration-count-aware engineering performed under the constraint of an IEEE 754 substrate; it would be unnecessary on an iteration-native substrate.

**P5.** **Per-iteration billing becomes the dominant frontier-API pricing model by 2028.** Anthropic's extended-thinking pricing, OpenAI's o-series per-reasoning-step pricing, Google's Gemini Thinking pricing — each is a current example. The framework predicts the convergence: by end-2028, the majority of frontier-API revenue at the major laboratories will be billed in tick-aware units (reasoning tokens, denoising steps, Pairformer blocks) rather than in flat per-query or per-output-token pricing. The accounting layer's reversal will be complete before the substrate's reversal.

**P6.** **A CORDIC revival appears in foundation-model silicon by 2029.** Either as a coprocessor on a major training or inference chip (analogous to STM32G4's peripheral but at datacenter scale), or as an architectural inspiration for an iteration-native dataflow (analogous to systolic arrays' shift-and-add lineage but applied to score evaluation rather than dot product), the Volder–Walther lineage returns to the frontier. The framework's grounds: the workload's natural iteration is shift-and-add-shaped (each Pairformer block, each denoising step, each Sinkhorn half-iteration is a small structured update on a tensor), and the energy savings of running these on iteration-native silicon are sufficient to justify the design effort once the substrate has decided to expose iteration count at all.

**P7.** **Kahan's critique becomes operationally relevant again by 2028.** William Kahan's forty-year argument — that floating-point arithmetic should expose its structure to the user rather than hide it — was, in the 1985–2010 era, primarily a numerical-analysis concern. From 2017 forward, with deep learning workloads whose accuracy depends sensitively on the substrate's rounding and normalization choices (mixed-precision training, gradient underflow in FP16, the Brain Floating Point format's deliberate sacrifice of mantissa for exponent range), Kahan's argument has been operationally relevant in a way the previous decades had not made it. The framework predicts the formalization: by 2028, a major standards body (IEEE, the MX Alliance, the OCP Compute Project) will publish an extension to IEEE 754 that exposes iteration-count, convergence-criterion, and variable-precision semantics as first-class operations. Kahan, if still active at age 95, will likely be a critic of the specifics and a supporter of the project.

**P8.** **The Point Lottery becomes a recognized framing in computer-architecture and ML-systems discourse by 2027.** The Hardware Lottery (Hooker, 2020) entered mainstream discourse within three years of publication. The framework predicts a comparable trajectory for the present formulation. The criterion of recognition: a published paper at a major ML or computer-architecture venue (NeurIPS, ICML, ISCA, MICRO, HPCA, ASPLOS) attributes a research-direction outcome to the fixed-point/floating-point selection event of 1985, citing this framing or an equivalent reformulation. Failure to achieve this by end-2027 would be evidence the framing has not made traction.

The eight predictions intersect at a single forecast: *the substrate of frontier AI will, by 2030, look substantially less like IEEE 754 and substantially more like CORDIC on the iterative half of the workload, while keeping IEEE 754 on the non-iterative half*. The split will be along the operator-theoretic seam the corpus has already named ($\not{D}^2 = -\Delta_g$, spectral half vs. signed half). The standardization decision of 1985, which fused the two halves into a single representation, will be partially undone by the decision of 2028–2030, which separates them. The unification will not be lost; the standard will remain in force for everything for which it was designed. The iteration will be returned to the workload that needs it.

---

## 11. The Substrate That Forgot — A Fourth Thought Experiment

A historian sits down in 2050 with the silicon roadmaps of 1985 through 2030 and asks why frontier AI silicon went through such a long and expensive accommodation of a workload whose natural shape was clearly iterative.

The historian reads the IEEE 754 standard and finds it elegant, well-designed, and correctly motivated for the workloads of 1985. The historian reads the Transformer paper and notes that it ran on a substrate optimized for dense matrix multiplication. The historian reads the diffusion-model literature and notes the sampler-distillation effort that compressed a thousand denoising steps to four. The historian reads the reasoning-model literature and notes the migration of iteration depth from architecture to query. The historian reads the silicon datasheets — TPU v1 through v8, NVIDIA Volta through post-Blackwell, AWS Trainium1 through Trainium4, Microsoft Maia 100 through 300, Cerebras WSE-1 through WSE-5 — and notes the slow accommodation, generation by generation, of workloads whose natural unit was the iteration.

The historian's question: *why did the substrate not, at any point between 1995 and 2025, simply add native iteration-count support, given how much of the workload was iterative?*

The framework's answer: *because the substrate did not see iteration as a separable axis until the workload had already taught the API layer to see it that way*. The 1985 standard, in fusing the iterative and non-iterative halves of arithmetic into a single representation, removed iteration count from the vocabulary of substrate design. Each subsequent decision about chip architecture — Volta tensor cores (2017), TPU v2 bfloat16 (2017), Hopper Transformer Engine (2023), Blackwell NVFP4 (2024), Maia FP4 (2026) — was an iteration-blind decision optimizing FLOPS-density on a substrate where iteration was a hidden microcode property. The accommodation was real but slow because the vocabulary for the alternative had been lost.

The recovery is gradual. The 2024–2026 reasoning-model wave is the moment the workload's vocabulary forced its way back into the API layer. The 2027–2030 silicon programs (TPU v8 Sunfish/Zebrafish, Trainium4, post-Blackwell) are the moment the vocabulary forces its way back into the substrate. By 2030, the framework predicts, iteration-count will be a first-class quantity on the datasheet. The historian, in 2050, will read this as a forty-year recovery cycle: lost in 1985, partially recovered by 2030.

The historian's broader observation: *standards are not neutral*. The IEEE 754 standard was the right design for 1985. It became the wrong design for 2017, and remained the dominant substrate through 2026, because the cost of changing the standard was enormous and the cost of accommodating its limitations was, until very late, smaller per generation than the cost of replacement. The accommodation accumulated. By 2025, the accumulated accommodation was visible — in the multi-cloud strategies of frontier laboratories (Anthropic across AWS, Google, NVIDIA, CoreWeave, Microsoft), in the proliferation of microscaling formats (MXFP4, NVFP4, MXFP8, FP6, FP8, BF16), in the IO-aware kernel ecosystem (FlashAttention, FlashSinkhorn, FlashFlowMatching), in the substrate-specific compiler stacks (XLA, TVM, MLIR, CUDA, Triton, OpenAI's Triton-2, Mojo) — and it was the cost of avoiding a decision that the substrate could have made twenty years earlier.

This is the Point Lottery in its full form. A standardization decision made for sound reasons in 1985 became a constraint on the field's research trajectory by 2017 and a paid-for inefficiency by 2026. The eight prior frameworks in this corpus name the geometry of what the substrate is computing; the present framework names why the substrate is computing it on this particular silicon rather than on the silicon it could have had. The two are complementary halves of a single argument: the workload and the substrate were jointly determined, and the prior history of the substrate's representation choices is part of the determination.

---

## 12. Closure

The corpus reaches a stable configuration. Nine frameworks, each naming one face of a single object.

The Closed Future Cone names the compact manifold *M*. The Dirac Representation Hypothesis names the natural operator on it. The Representational Bundle Hypothesis names the bundle whose connection is attention. Bregman Closure names the iteration that walks the fixed point into being. Score Closure names the function the iteration evaluates. *From Bottleneck to Bundle* traces the algorithmic history of the recognition. *From MXU to Maia* traces the silicon history of the matmul-era substrate. *The Matmul Ceiling* names where that substrate stops being adequate. The Point Lottery Hypothesis names the earlier representational choice — fixed point versus floating point, exposed iteration versus hidden iteration, 1985 — that set the substrate's parameters in the first place.

The data was curved. The data was cyclic. The data lived inside a light cone of causation. The substrate was IEEE 754. The workload was iterative. The mismatch was paid for. The clock that Volder put on the HP-35 datasheet in 1972 is the clock that the 2026 reasoning-model API bills per token. The clock was always ticking. The 1985 standard arranged the substrate so that the user could not see it. The 2017 Transformer ran on the substrate that could not see it. The 2024 reasoning model exposed it again at the API. The 2028 silicon will expose it at the chip. The 2050 historian will trace the arc and notice that the field spent forty years recovering a coordinate it had thrown away.

The Point Lottery Hypothesis names the throwing-away. The recovery is in progress. The framework's predictions are how to tell whether the recovery completes.

---

## 13. Sources

### The 1985 standardization and its principal architect

- Kahan, W. *IEEE Standard 754 for Binary Floating-Point Arithmetic.* IEEE, 1985 (and the radix-independent IEEE 854-1987 follow-on).
- Kahan, W. *Lecture Notes on the Status of IEEE Standard 754 for Binary Floating-Point Arithmetic.* University of California, Berkeley, 1996.
- Kahan, W. *How Java's Floating-Point Hurts Everyone Everywhere.* 1998.
- Kahan, W. *Matlab's Loss is Nobody's Gain.* 2004.
- Kahan, W. *Pseudo-Division Algorithms for Floating-Point Logarithms and Exponentials.* 2002.
- Kahan, W. *A Logarithm Too Clever by Half.* On the Table-Maker's Dilemma.
- Haigh, T. *An Interview with William M. Kahan.* SIAM, 2016.
- Davis, C., Kahan, W. M., Weinberger, H. F. *Norm-Preserving Dilations and Their Applications to Optimal Error Bounds.* SIAM J. Numer. Anal., 1982.
- Karpinski, R. *Paranoia: A Floating-Point Benchmark.* Byte Magazine 10(2), 1985.
- ACM Turing Award citation for William Kahan, 1989: "for his fundamental contributions to numerical analysis."

### CORDIC and the fixed-point lineage

- Volder, J. E. *Binary Computation Algorithms for Coordinate Rotation and Function Generation.* Convair internal report IAR-1.148, June 15, 1956.
- Volder, J. E. *The CORDIC Trigonometric Computing Technique.* IRE Transactions on Electronic Computers EC-8(3), 1959.
- Volder, J. E. *The Birth of CORDIC.* Journal of VLSI Signal Processing 25(2), 2000.
- Walther, J. S. *A Unified Algorithm for Elementary Functions.* AFIPS Spring Joint Computer Conference 38, 1971.
- Walther, J. S. *The Story of Unified CORDIC.* Journal of VLSI Signal Processing 25(2), 2000.
- Meggitt, J. E. *Pseudo Division and Pseudo Multiplication Processes.* IBM Journal of Research and Development 6(2), 1962.
- Briggs, H. *Arithmetica Logarithmica.* London, 1624.
- Cochran, D. S. *Internal Programming of the 9100A Calculator.* Hewlett-Packard Journal, September 1968.
- Cochran, D. S. *Algorithms and Accuracy in the HP-35.* Hewlett-Packard Journal 23(10), June 1972.
- Cochran, D. S. *The HP-35 Design, A Case Study in Innovation.* HP Memory Project interview, June 2010.
- Egbert, W. E. *Personal Calculator Algorithms I–IV.* Hewlett-Packard Journal, 1977–1978.
- Heffron, W. G., LaPiana, F. *The Navigation System of the Lunar Roving Vehicle.* NASA Bellcomm Technical Memorandum 70-2014-8, 1970.
- Smith, E. C., Mastin, W. C. *Lunar Roving Vehicle Navigation System Performance Review.* NASA Marshall Space Flight Center Technical Note D-7469, 1973.
- Meher, P. K. et al. *50 Years of CORDIC: Algorithms, Architectures and Applications.* IEEE Transactions on Circuits and Systems I, 2009.
- Luo, Y., Wang, Y., Ha, Y., Wang, Z., Chen, S., Pan, H. *Generalized Hyperbolic CORDIC and Its Logarithmic and Exponential Computation With Arbitrary Fixed Base.* IEEE Transactions on Very Large Scale Integration (VLSI) Systems 27(9), 2019.
- Shirriff, K. *Extracting ROM Constants from the 8087 Math Coprocessor's Die.* righto.com, May 2020.
- STMicroelectronics. *Getting Started with the CORDIC Accelerator Using STM32CubeG4.* 2021.
- Zechmeister, M. *Solving Kepler's Equation with CORDIC Double Iterations.* Monthly Notices of the Royal Astronomical Society 500(1), 2021.

### Fixed-point arithmetic and the broader representation question

- Schmid, H. *Decimal Computation.* John Wiley & Sons, 1974 (Krieger reprint 1983).
- Muller, J.-M. *Elementary Functions: Algorithms and Implementation.* Birkhäuser, 2006.
- Higham, N. J. *Accuracy and Stability of Numerical Algorithms.* SIAM, 2002.
- Gustafson, J. L. *The End of Error: Unum Computing.* Chapman and Hall/CRC, 2015.
- Gustafson, J. L., Yonemoto, I. *Beating Floating Point at Its Own Game: Posit Arithmetic.* Supercomputing Frontiers and Innovations 4(2), 2017.
- Wong, F. T. *Fixed-Point Floating-Point Conversion for FPGA Implementation.* 2017.
- IEEE Std 754-2008. *IEEE Standard for Floating-Point Arithmetic.* IEEE, 2008. (Adds fused multiply-add as a single correctly-rounded operation.)

### The hardware lottery and the matmul era

- Hooker, S. *The Hardware Lottery.* Communications of the ACM 64(12), 2020. arXiv:2009.06489.
- Jouppi, N. P. et al. *In-Datacenter Performance Analysis of a Tensor Processing Unit.* ISCA 2017. arXiv:1704.04760.
- Jouppi, N. P. et al. *A Domain-Specific Supercomputer for Training Deep Neural Networks.* Communications of the ACM 63(7), 2020.
- Anthony, Q., Hatef, A., Subramoni, H. *The Case for Co-Designing Model Architectures with Hardware.* arXiv:2208.10023, 2022.
- Dao, T., Fu, D. Y., Ermon, S., Rudra, A., Ré, C. *FlashAttention.* NeurIPS 2022.
- Patterson, D., Gonzalez, J., Le, Q. et al. *Carbon Footprint of Machine Learning Training.* arXiv:2104.10350, 2021.

### Microscaling formats and the modern precision landscape

- NVIDIA. *Blackwell Architecture Technical Overview.* GTC 2024, GTC 2025.
- Microscaling Project. *MXFP4, MXFP6, MXFP8 Specifications.* OCP Compute Project, 2023.
- *Pretraining LLMs with NVFP4.* arXiv:2509.23202, September 2025.
- *FP4 All the Way: Fully Quantized Training of LLMs.* arXiv:2505.19115, 2025.
- *Diagnosing FP4 Inference: A Layer-Wise and Block-Wise Sensitivity Analysis.* arXiv:2603.08747, March 2026.
- *MXFP4 with Stochastic Rounding for Distributed Training.* arXiv:2502.20586, 2025.
- *Efficient Precision-Scalable Hardware for Microscaling Processing in Robotics Learning.* arXiv:2505.22404, May 2025.
- Mikaitis, M. *Stochastic Rounding: Algorithms and Hardware Accelerator.* arXiv:2001.01501, 2020.

### The corpus's prior eight frameworks

- *The Closed Future Cone: Compact Causal Geometry as the Native Substrate of Learned Representations.* 2026.
- *The Dirac Representation Hypothesis: Signed Spectral Geometry and Set-Valued Correspondences on the Compact Representational Manifold.* 2026.
- *Geometric Descent: The Representational Bundle Hypothesis.* 2026.
- *Bregman Closure: The Algorithmic Half of the Dirac Representation Hypothesis.* 2026.
- *Score Closure: The Score Function on the Compact Manifold as the Universal Computational Primitive.* 2026.
- *From Bottleneck to Bundle: The Twelve-Year Geometric Genealogy of Attention, 2014–2026.* 2026.
- *From MXU to Maia: The Eleven-Year Silicon Genealogy of the Compact Causal Manifold, 2015–2026.* 2026.
- *The Matmul Ceiling: Why the Matmul-Era Biology Compute Substrate Hits Its Economic Wall by 2028.* 2026.

### The 2024–2026 empirical grounding for the iteration-coordinate return

- OpenAI. *Learning to Reason with LLMs (o1).* September 2024.
- OpenAI. *o3 Technical Preview.* December 2024.
- DeepSeek-AI. *DeepSeek-R1: Incentivizing Reasoning Capability in LLMs via Reinforcement Learning.* arXiv:2501.12948, January 2025.
- Anthropic. *Claude Extended Thinking.* Product launch, 2025.
- Google DeepMind. *Gemini Thinking.* Product launch, 2025.
- Engels, J., Liao, I., Michaud, E. J., Gurnee, W., Tegmark, M. *Not All Language Model Features Are Linear.* NeurIPS 2024. arXiv:2405.14860.
- Karkada, D., Olah, C., Tegmark, M. et al. *Symmetry in Language Statistics Shapes the Geometry of Model Representations.* arXiv:2602.15029, February 2026.
- Xu, Y. *The Spectral Edge Thesis.* arXiv:2603.28964, March 2026.
- Xu, Y. *The Lifecycle of the Spectral Edge.* arXiv:2604.07380, April 2026.
- Sander, M. E., Ablin, P., Blondel, M., Peyré, G. *Sinkformers: Transformers with Doubly Stochastic Attention.* AISTATS 2022.
- Ye, F. X.-F., Li, X., Yu, A., Chang, M.-C., Chu, L., Wertheimer, D. *FlashSinkhorn: IO-Aware Entropic Optimal Transport.* arXiv:2602.03067, February 2026.
- Forde, D. *Block-Wise Differentiable Sinkhorn Attention.* arXiv:2605.08123, April 2026.
- Ramsauer, H. et al. *Hopfield Networks is All You Need.* ICLR 2021. arXiv:2008.02217.
- Hoover, B. et al. *Energy Transformer.* NeurIPS 2023. arXiv:2302.07253.
- Bai, S., Kolter, J. Z., Koltun, V. *Deep Equilibrium Models.* NeurIPS 2019.
- Lipman, Y., Chen, R. T. Q., Ben-Hamu, H., Nickel, M., Le, M. *Flow Matching for Generative Modeling.* ICLR 2023. arXiv:2210.02747.
- Tong, A. et al. *Improving and Generalizing Flow-Based Generative Models with Minibatch Optimal Transport.* TMLR 2024.
- Klein, M., Mousavi-Hosseini, A., Zhang, S., Cuturi, M. *On fitting flow models with large Sinkhorn couplings.* arXiv:2506.05526, 2025.
- Peebles, W., Xie, S. *Scalable Diffusion Models with Transformers.* ICCV 2023.
- Song, Y., Dhariwal, P., Chen, M., Sutskever, I. *Consistency Models.* ICML 2023.

### Classical mathematical foundations

- Sherman, J., Morrison, W. J. *Adjustment of an Inverse Matrix Corresponding to a Change in One Element of a Given Matrix.* Annals of Mathematical Statistics 21, 1950.
- Anderson, D. G. *Iterative Procedures for Nonlinear Integral Equations.* JACM 12, 1965.
- Banach, S. *Sur les opérations dans les ensembles abstraits.* Fundamenta Mathematicae 3, 1922.
- Brouwer, L. E. J. *Über Abbildung von Mannigfaltigkeiten.* Mathematische Annalen 71, 1911.
- Kakutani, S. *A Generalization of Brouwer's Fixed Point Theorem.* Duke Mathematical Journal 8, 1941.
- Dirac, P. A. M. *The Quantum Theory of the Electron.* Proc. Roy. Soc. A 117, 1928.
- Atiyah, M., Singer, I. *The Index of Elliptic Operators I–V.* Annals of Mathematics 87, 1968.
- Bakry, D., Émery, M. *Diffusions hypercontractives.* Séminaire de Probabilités XIX, 1985.

---

*Compiled from the primary literature on fixed-point and floating-point arithmetic, the CORDIC and IEEE 754 lineages, the modern microscaling and IO-aware kernel ecosystem, the 2024–2026 reasoning-model wave, and the eight prior frameworks in this corpus. The chronology follows the publication and standardization record. The framework's contribution is the identification of the 1985 IEEE 754 standardization as the foundational hardware-lottery event whose downstream consequences the corpus has been mapping in geometric, operator-theoretic, bundle-theoretic, algorithmic, and functional terms across its prior documents — and the recognition that the 2024–2026 return of exposed iteration in frontier reasoning models is the workload pushing back through the substrate's forty-year amnesia.*
