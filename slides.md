

<div style="text-align: center; margin-top: -30px; margin-bottom: 12px;">
  <img src="/src/icon.svg" style="height: 30px; display: inline-block;" />
</div>

<h1 style="font-size: 3.1rem; line-height: 1.3; text-align: center;">SCas4D: Structural Cascaded Optimization for Boosting Persistent 4D Novel View Synthesis</h1>

<h1 style="font-size: 1.2rem; line-height: 1.3; text-align: center;">Jipeng Lyu, Jiahua Dong, Yu-Xiong Wang</h1>
<h1 style="font-size: 1.2rem; line-height: 0.3; text-align: center;">University of Illinois Urbana-Champaign</h1>

<!--
Hello everyone, I am very happy to present our work SCas4D.
In this talk, I will show how adding structure to dynamic Gaussian optimization
makes online 4D novel view synthesis much faster and more stable.
-->

---

# Online 4DGS: Fast Adaptation

<div class="text-left text-sm">

- Each frame starts from the previous one
- Motion can be large and highly non-rigid
- Low iteration budget makes optimization hard

</div>


<div class="flex" style="width: 100%; margin-top: 16px;">
  <div style="width: 30%;">
    <img src="/src/video_output/gt_robot_3.gif" style="width: 100%; display: block;" />
  </div>
  <div style="width: 30%;">
    <img src="/src/video_output/gt_cloth_0.gif" style="width: 100%; display: block;" />
  </div>
  <div style="width: 40%; display: flex; align-items: center;">
    <img src="/src/video_output/gt_football_1.gif" style="width: 100%; display: block;" />
  </div>
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

# Problem: Optimization Instability

<div class="text-left text-sm">

- Standard methods update Gaussians independently
- Under small budget, optimization drifts or collapses
- Worsens for large motion and non-rigid deformation

</div>


<div class="flex" style="width: 100%; height: 38vh; margin-top: 16px;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_cloth_0.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_cloth_0.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0; display: flex; align-items: center;">
      <img src="/src/video_output/gt_cloth_0.gif" style="width: 100%; height: auto; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
The challenge is not whether dynamic Gaussians can eventually fit the scene,
but whether they can fit it correctly with very few iterations per frame.
Under this online setting, independent per-Gaussian updates often drift or break down,
especially for large motion and non-rigid deformation.
-->

---

# Key Insight: Structured Deformation

<div class="text-left text-sm">

- Nearby Gaussians share motion patterns
- Large structures move first, details refined later
- Coarse-to-fine instead of independent per-point updates

</div>

<img src="/src/K1_0.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
The core insight is that dynamic deformation is not arbitrary.
Many Gaussians share motion patterns,
so it is more natural to move them as groups first,
and then refine locally.
This gives us a structured optimization problem
instead of a fully unstructured one.
-->

---

# Key Insight: Structured Deformation

<div class="text-left text-sm">

- Nearby Gaussians share motion patterns
- Large structures move first, details refined later
- Coarse-to-fine instead of independent per-point updates

</div>

<img src="/src/K1_1.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
The core insight is that dynamic deformation is not arbitrary.
Many Gaussians share motion patterns,
so it is more natural to move them as groups first,
and then refine locally.
This gives us a structured optimization problem
instead of a fully unstructured one.
-->

---

# Method: SCas4D

<div class="text-left text-sm">

- Organize Gaussians into **K-layer clusters**
- Each layer has a deformation function
- Compose from coarse to fine

</div>

<img src="/src/P2.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
Starting from the Gaussians of the previous frame,
we organize them into a multi-layer cluster hierarchy.
Each layer predicts a deformation,
and the full scene deformation is the composition of these layers.
The whole model is supervised through the rendered image loss.
-->

---

# Cluster Deformation

<div class="text-left text-sm">
<!-- 
For each cluster, we model: -->

- Rotation
- Translation
- Position-dependent scaling

</div>

<img src="/src/P2.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
A cluster deformation is parameterized in an interpretable way:
rotation, translation, and position-dependent scaling.
This is expressive enough to capture major motion patterns,
while remaining much easier to optimize
than unconstrained per-point motion.
-->

---

# Coarse-to-Fine Hierarchy

<div class="text-left text-sm">

- Coarse layers align large structures quickly
- Fine layers refine smaller local motion
- Per-Gaussian residuals preserve detail

</div>

<img src="/src/P2.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
An important point is that we do not lose fine-grained modeling power.
The coarse layers bring the scene quickly to the right region,
while finer layers and per-Gaussian residuals preserve local detail.
So the hierarchy accelerates optimization
without over-constraining the representation.
-->

---

# End-to-End via Rendering

<div class="text-left text-sm">

- Deform Gaussians for the current frame
- Render the image
- Backpropagate image loss

</div>

<img src="/src/G1.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<!--
After deformation, we render the current frame
and optimize all deformation parameters through the image loss.
In this sense, SCas4D fits naturally into the standard online 4DGS pipeline,
but changes the deformation parameterization
into a much more structured one.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Robot</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_robot_3.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_robot_3.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_robot_3.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Cloth</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_cloth_0.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_cloth_0.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_cloth_0.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Spring</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_spring_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_spring_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_spring_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Football</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_football_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_football_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_football_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Boxes</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_boxes_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_boxes_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_boxes_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Results: Quality at 100 it/frame

<div class="text-center font-semibold mt-10 mb-6" style="font-size: 1.1rem;">Softball</div>

<div class="flex" style="width: 100%; height: 38vh;">
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ours</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/ours_softball_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Dynamic3DGS</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/deva_softball_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
  <div class="flex flex-col" style="width: 33.33%;">
    <div class="text-center" style="font-size: 0.95rem; line-height: 0.;">Ground Truth</div>
    <div style="flex: 1; min-height: 0;">
      <img src="/src/video_output/gt_softball_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
    </div>
  </div>
</div>

<div style="position: absolute; bottom: 16px; left: 24px; right: 24px; font-size: 0.6rem; color: #888;">
Luiten, Jonathon, et al. "Dynamic 3d gaussians: Tracking by persistent dynamic view synthesis." arXiv preprint arXiv:2308.09713 (2023).
</div>

<!--
Under the same low iteration budget of 100 iterations per frame,
our method is consistently more stable than the baseline.
-->

---

# Better Tracking

<div class="text-left text-sm">

- More stable and coherent motion
- More accurate dense point trajectories

</div>

<img src="/src/T2.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<div class="text-center mt-2" style="font-size: 0.8rem; color: #555;">
Left: Comparing our tracking result (blue) to the ground truth (red). Right: Visualization of our tracking results.
</div>

<!--
Because our deformation field is more stable and more coherent,
it is also useful for downstream motion analysis.
In the paper we show that this leads to better dense point tracking as well.
-->

---

# Self-supervised Segmentation

<div class="text-left text-sm">

- Cluster motion across time
- Recover motion-consistent parts
- No semantic labels required

</div>

<div class="flex items-center" style="width: 100%; margin-top: 20px; height: 42vh;">
  <div style="width: 38%; height: 100%; display: flex; align-items: center;">
    <img src="/src/video_output/gt_boxes_1.gif" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
  </div>
  <div style="width: 24%; display: flex; align-items: center; justify-content: center; font-size: 2.5rem; color: #444;">
    <span style="display: inline-block; transform: scaleX(3); transform-origin: center;">→</span>
  </div>
  <div style="width: 38%; height: 100%; display: flex; align-items: center;">
    <img src="/src/S1_boxes.png" style="width: 100%; height: 100%; object-fit: contain; display: block;" />
  </div>
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

- Independent → **structured cascaded deformation**
- Faster convergence under low iteration budget
- More stable rendering quality
- Motion cues for tracking & segmentation

</div>

<img src="/src/C1.png" style="display: block; width: 72%; margin: 16px auto 0;" />

<div class="text-center mt-4" style="font-size: 0.95rem;">
  Please check out our paper for more details —
  <a href="https://arxiv.org/pdf/2510.06694" target="_blank" style="color: #4a90d9;">arxiv.org/pdf/2510.06694</a>
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
