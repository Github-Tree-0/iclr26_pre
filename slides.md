# SCas4D: Structural Cascaded Optimization for Boosting Persistent 4D Novel View Synthesis

Jipeng Lyu, Jiahua Dong, Yu-Xiong Wang  
University of Illinois Urbana-Champaign

<div class="text-left text-sm mt-4">

**Structured dynamic optimization for fast and stable 4DGS**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #T1]</b><br><br>
Use a clean teaser crop or one strong visual from the paper.

</div>

<!--
Hello everyone, thank you for coming.
I am very happy to present our work SCas4D.
In this talk, I will show how adding structure to dynamic Gaussian optimization
makes online 4D novel view synthesis much faster and more stable.
-->

---

# Online 4DGS Needs to Adapt Fast

<div class="text-left text-sm">

- each frame starts from the previous one
- motion can be large and highly non-rigid
- low iteration budget makes optimization hard

</div>

<div class="text-left text-base mt-4">

**Can we make dynamic 4DGS converge faster and more stably?**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Video Placeholder #M1]</b><br><br>
Short motivating clip of a challenging dynamic scene.  
No comparison needed yet.

</div>

<!--
Our setting is online persistent 4D novel view synthesis.
At each frame, we inherit the Gaussians from the previous frame
and need to adapt them quickly to new observations.
This is challenging because motion can be large,
deformation can be complex,
and the optimization budget per frame is very limited.
-->

---

# Problem: Low-Budget Online Optimization is Unstable

<div class="text-left text-sm">

- standard methods update Gaussians largely independently
- under a small budget, optimization can drift or collapse
- this becomes worse for large motion and non-rigid deformation

</div>

<div class="text-left text-base mt-4">

**Low budget → drift, collapse, and poor rendering quality**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Video Placeholder #P1]</b><br><br>
Failure case under 100 it/frame.  
Recommended scenes: Robot or Cloth.  
Show Ours vs Dynamic3DGS vs GT if possible.

</div>

<!--
The challenge is not whether dynamic Gaussians can eventually fit the scene,
but whether they can fit it correctly with very few iterations per frame.
Under this online setting, independent per-Gaussian updates often drift or break down,
especially for large motion and non-rigid deformation.
-->

---

# Key Insight: Deformation is Structured

<div class="text-left text-sm">

Instead of updating each Gaussian independently,  
we optimize motion in a **coarse-to-fine structured way**.

- nearby Gaussians often move together
- large structures should move first
- local details can be refined later

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #K1]</b><br><br>
Concept figure:  
Left = independent per-Gaussian motion  
Right = hierarchical clustered motion from coarse to fine

</div>

<!--
The core insight is that dynamic deformation is not arbitrary.
Many Gaussians share motion patterns,
so it is more natural to move them as groups first,
and then refine locally.
This gives us a structured optimization problem
instead of a fully unstructured one.
-->

---

# SCas4D: Structural Cascaded Optimization

<div class="text-left text-sm">

- organize Gaussians into **K-layer clusters**
- each layer has a deformation function
- compose them from coarse to fine

</div>

<div class="text-left text-base mt-4">

**D = D₁ ∘ D₂ ∘ ... ∘ Dₖ**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #P2]</b><br><br>
Pipeline overview:  
previous-frame Gaussians → K-layer cluster structure → deformation → rendering → backpropagation

</div>

<!--
Starting from the Gaussians of the previous frame,
we organize them into a multi-layer cluster hierarchy.
Each layer predicts a deformation,
and the full scene deformation is the composition of these layers.
The whole model is supervised through the rendered image loss.
-->

---

# Interpretable Cluster Deformation

<div class="text-left text-sm">

For each cluster, we model:

- rotation
- translation
- position-dependent scaling

</div>

<div class="text-left text-base mt-4">

**Stable, expressive, and easy to optimize**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #D1]</b><br><br>
Use the figure that visualizes rotation / translation / scaling on one cluster.

</div>

<!--
A cluster deformation is parameterized in an interpretable way:
rotation, translation, and position-dependent scaling.
This is expressive enough to capture major motion patterns,
while remaining much easier to optimize
than unconstrained per-point motion.
-->

---

# Coarse Structure + Fine Residual Freedom

<div class="text-left text-sm">

- coarse layers align large structures quickly
- fine layers refine smaller local motion
- per-Gaussian residuals preserve detail

</div>

<div class="text-left text-base mt-4">

**Fast convergence without sacrificing fidelity**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #H1]</b><br><br>
Hierarchy visualization across layers, or a custom three-level diagram.

</div>

<!--
An important point is that we do not lose fine-grained modeling power.
The coarse layers bring the scene quickly to the right region,
while finer layers and per-Gaussian residuals preserve local detail.
So the hierarchy accelerates optimization
without over-constraining the representation.
-->

---

# End-to-End Optimization Through Rendering

<div class="text-left text-sm">

- deform Gaussians for the current frame
- render the image
- backpropagate image loss

</div>

<div class="text-left text-base mt-4">

**Minimal change to the online 4DGS pipeline**

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #G1]</b><br><br>
Pipeline page with gradient flow annotation.

</div>

<!--
After deformation, we render the current frame
and optimize all deformation parameters through the image loss.
In this sense, SCas4D fits naturally into the standard online 4DGS pipeline,
but changes the deformation parameterization
into a much more structured one.
-->

---

# Main Result: Much Better Quality at 100 it/frame

<div class="text-left text-sm">

- clearly better than Dynamic3DGS under the same budget
- close to the quality of much longer optimization
- more stable structure and less drift

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Video Placeholder #R1]</b><br><br>
Main qualitative comparison.  
Recommended: one synthetic scene + one real scene.  
Examples: Cloth + Football, or Robot + Boxes.

</div>

<!--
This is the main takeaway.
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
Qualitatively, we see better structure preservation,
less drift, and cleaner deformation.
-->

---

# Quantitative Result: 100 it/frame is Enough

<div class="text-left text-sm">

- outperform baseline at the same iteration budget
- approach the quality of much longer baseline optimization
- significant iteration-level convergence speedup

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #Q1]</b><br><br>
Clean crop of the main quantitative table.  
Optional: add the convergence curve as a side panel.

</div>

<!--
Quantitatively, SCas4D significantly outperforms the baseline
under the same 100-iteration budget,
and approaches the quality reached by much longer baseline optimization.
So the key claim is not just better final quality,
but much faster convergence in the online setting.
-->

---

# Extra Benefit: Better Tracking

<div class="text-left text-sm">

- more stable and coherent motion
- more accurate dense point trajectories

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #T2]</b><br><br>
Tracking figure with trajectory comparison.

</div>

<!--
Because our deformation field is more stable and more coherent,
it is also useful for downstream motion analysis.
In the paper we show that this leads to better dense point tracking as well.
-->

---

# Extra Benefit: Self-supervised Articulated Segmentation

<div class="text-left text-sm">

- cluster motion across time
- recover motion-consistent parts
- no semantic labels required

</div>

<div style="width: 100%; height: 300px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Video Placeholder #S1]</b><br><br>
Segmentation result from the supplemental material.

</div>

<!--
Another nice byproduct is that the learned deformation parameters
are informative by themselves.
By clustering motion features over time,
we can recover articulated parts in a self-supervised way,
which gives a useful segmentation signal without semantic annotation.
-->

---

# Takeaway

<div class="text-left text-sm">

SCas4D turns online 4DGS optimization from  
**independent point updates** into  
**structured cascaded deformation**.

</div>

<div class="text-left text-sm mt-4">

This gives:

- faster convergence
- more stable rendering
- useful motion cues for downstream tasks

</div>

<div class="text-left text-base mt-4">

**Please check out our paper for more details.**

</div>

<div style="width: 100%; height: 240px; border: 2px dashed rgb(102,102,102); padding: 16px; border-radius: 8px; overflow: auto; margin-top: 20px;">

<b>[Image Placeholder #C1]</b><br><br>
Closing summary visual.  
Option A: reuse teaser image  
Option B: three thumbnails for rendering, tracking, segmentation

</div>

<!--
To conclude, the main contribution of SCas4D
is to turn unstructured dynamic Gaussian optimization
into a structured cascaded process.
This simple change gives faster convergence,
better stability,
and useful motion features for downstream tasks.
Thank you very much,
and please check out our paper for more details.
-->