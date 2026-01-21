# Adaptive Tiled Efficient Splatting

Apart from the original Speedy Splat implementation we made the following changes:
- `scene/gaussian_model.py`: Records the following during training:
  - `gaussian_end_parameters.csv`: The parameters at the end of the life of all gaussians (identified with an ID).
  - `gaussian_start_parameters.csv`: The parameters at the beginning of all gaussians
  - `gaussian_lifecycle.csv`: The creation time/iteration of all gaussians and their end time/iteration giving us the lifetime in iterations and wall clock
  - `gaussian_spawned.csv`: A table of parent-child relationships, where a parent is a gaussian, which caused the creation of another one (child)
- `train.py`: Records the final metrics to `final_metrics.csv`
- `gaussian_analysis.py`: Plots the lifecycle distribution and puts the gaussians into two categories: Parents and childless
- `scatter.py`: Creates scatter plots of pairs of parameters and also separates into the two categories.
- `heatmap.py`: Projects the gaussian blobs, which we consider useless, to the cameras and marks their position in the reference images.
- `go.sh`: Simply a script to run the training on all inputs

Our hypothesis is that gaussians which are childless and die before the last iteration are unnecessary for the learning and just increase computational cost.
We are trying to find a heuristic which would separate this group at the time of creation from the rest, in order to prevent their creation.
That's why we first analysed how many such blobs exist (`gaussian_analysis.py`), which turns out to be the majority.
Then we tried to infer a rule with the parameters using the scatter plot. No clear distinction could be observed.
Lastly, we projected the useless blobs as a heatmap onto the cameras and the reference images to see if we could come up with a heuristic based on the reference image we are trying to fit.


## TODO
The heatmaps look like they are shifted. We should check on simple examples if they are correct, meaning writing a simple camera image setup with a handful of blob parameters and see if they end up where we expect them to. Assuming the projection in `heatmap.py` is incorrect and the useless blobs would align nicely with the edges we need to experiment with edge detection algorithms (i.e. canny or difference of gaussians) and prevent densification near the spikes.


## Sources
- Speedy Splat: https://github.com/j-alex-hanson/speedy-splat
- Viewer: https://antimatter15.com/splat/
- Inputs: https://repo-sam.inria.fr/fungraph/3d-gaussian-splatting/datasets/input/tandt_db.zip



---


# Speedy-Splat: Fast 3D Gaussian Splatting with Sparse Pixels and Sparse Primitives

[Alex Hanson](https://www.cs.umd.edu/~hanson/), [Allen Tu](https://tuallen.github.io), [Geng Lin](https://www.cs.umd.edu/people/geng), [Vasu Singla](https://vasusingla.github.io/), [Matthias Zwicker](https://www.cs.umd.edu/~zwicker/), [Tom Goldstein](https://www.cs.umd.edu/~tomg/)

[arXiv](https://arxiv.org/abs/2412.00578) | [Website](https://speedysplat.github.io)

<img src="assets/speedy_splat_cover.png" alt="Speedy Splat Cover" />

This is the official code repository for the paper "Speedy-Splat: Fast 3D Gaussian Splatting with Sparse Pixels and Sparse Primitives". It is built from the [3D-GS](https://github.com/graphdeco-inria/gaussian-splatting) repository. If you make use of this code in your research, please consider citing this work in addition to ours.


**Abstract:** *3D Gaussian Splatting (3D-GS) is a recent 3D scene reconstruction technique that enables real-time rendering of novel views by modeling scenes as parametric point clouds of differentiable 3D Gaussians.  However, its rendering speed and model size still present bottlenecks, especially in resource-constrained settings.  In this paper, we identify and address two key inefficiencies in 3D-GS, achieving substantial improvements in rendering speed, model size, and training time.  First, we optimize the rendering pipeline to precisely localize Gaussians in the scene, boosting rendering speed without altering visual fidelity.  Second, we introduce a novel pruning technique and integrate it into the training pipeline, significantly reducing model size and training time while further raising rendering speed.  Our Speedy-Splat approach combines these techniques to accelerate average rendering speed by a drastic 6.71x across scenes from the Mip-NeRF 360, Tanks \& Temples, and Deep Blending datasets  with 10.6x fewer primitives than 3D-GS.*

## Setup
Follow the setup instructions for the original [3D-GS](https://github.com/graphdeco-inria/gaussian-splatting) codebase. Our code changes are made in (1) the differential renderer submodule and (2) the Python files in this repo.

## Running

To train and evaluate a scene with Speedy-Splat, set the environment variables `SCENE_DATA_PATH` and `SCENE_MODEL_PATH`, then run:

```shell
bash train.sh
```

* `SCENE_DATA_PATH` is the path to the COLMAP or NeRF Synthetic dataset.

* `SCENE_MODEL_PATH` is the path where the model will be saved.

This script follows the 3D-GS pipeline to train a scene on the data provided by `SCENE_DATA_PATH` and saves the model in the `SCENE_MODEL_PATH` directory. Tensorboard logging has been updated to include all metrics reported in the Speedy-Splat manuscript.

### Scene Metrics

To compute scene metrics for an already trained model, set the environment variables `SCENE_DATA_PATH`,`SCENE_MODEL_PATH`, and `ONLY_RAW_KERNEL_TIMES` then run:

```shell
bash compute_scene_metrics.sh
```

* `SCENE_DATA_PATH` is the path to the COLMAP or NeRF Synthetic dataset.

* `SCENE_MODEL_PATH` is the path to the pretrained 3D-GS model.

* `ONLY_RAW_KERNEL_TIMES` is a boolean (`true`|`false`) variable. When true, the kernel only returns raw kernel time per image.

This script returns all metrics reported in Speedy-Splat for a pretrained model located in `SCENE_MODEL_PATH` with corresponding data in `SCENE_DATA_PATH`. The metrics will be written to a CSV file located in `SCENE_MODEL_PATH/<train|test>/ours_<iteration>/metrics.csv`, where `train`|`test` is the corresponding dataset split and `iteration` is the model checkpoint iteration.

If `ONLY_RAW_KERNEL_TIMES` is set to `true`. then the script will generate a `kernel_times.csv` file instead in the same directory as `metrics.csv`. Each row in the CSV records the raw kernel time to render the corresponding image in milliseconds. Currently, this script sequentially renders and records each image in the dataset, then repeats this process 20 times.

## Results
<img src="assets/speedy_splat_comparison.png" alt="Speedy Splat Comparison"/>

<section class="section" id="BibTeX">
  <div class="container is-max-desktop content">
    <h2 class="title">BibTeX</h2>
    <pre><code>@InProceedings{HansonSpeedy,
    author    = {Hanson, Alex and Tu, Allen and Lin, Geng and Singla, Vasu and Zwicker, Matthias and Goldstein, Tom},
    title     = {Speedy-Splat: Fast 3D Gaussian Splatting with Sparse Pixels and Sparse Primitives},
    booktitle = {Proceedings of the Computer Vision and Pattern Recognition Conference (CVPR)},
    month     = {June},
    year      = {2025},
    pages     = {21537-21546},
    url       = {https://speedysplat.github.io/}
}</code></pre>
  </div>
  <h2 class="title">Related Work</h2>
  <p>For additional papers on efficient 3D Gaussian Splatting, see our group’s related work below. If your
      research builds on ours, we encourage you to cite these papers.</p>
  <ol>
      <li>
          <a href="https://pup3dgs.github.io/" target="_blank" rel="noopener">
              <strong>PUP-3DGS</strong>
          </a>
          <em>(CVPR 2025)</em>
          <small><a href="https://pup3dgs.github.io/#BibTeX" target="_blank">[BibTeX]</a></small>
          — Prune 90% of primitives from any pretrained 3D Gaussian Splatting model using a mathematically
          principled sensitivity score, more than tripling rendering speed while retaining more salient
          foreground information and higher visual fidelity than previous techniques at a substantially
          higher compression ratio.
      </li>
      <li>
          <a href="https://speede3dgs.github.io/" target="_blank" rel="noopener">
              <strong>SpeeDe3DGS</strong>
          </a>
          <small><a href="https://speede3dgs.github.io/#BibTeX" target="_blank">[BibTeX]</a></small>
          — Boost DeformableGS rendering speed from 20 to 276 FPS using temporal sensitivity pruning and
          groupwise SE(3) motion distillation, all while preserving the superior image quality of per-Gaussian
          neural motion.
      </li>
  </ol>
</section>


## Funding and Acknowledgments

This work was made possible by the IARPA WRIVA Program, the ONR MURI program, and DAPRA TIAMAT. Commercial support was provided by Capital One Bank, the Amazon Research Award program, and Open Philanthropy. Further support was provided by the National Science Foundation (IIS-2212182), and by the NSF TRAILS Institute (2229885). Zwicker was additionally supported by the National Science Foundation (IIS-2126407).
