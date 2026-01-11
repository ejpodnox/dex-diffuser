# Troubleshooting

## Installation Issues

### `ModuleNotFoundError: No module named 'pytorch_kinematics'`
This package is required for robot kinematics but might be missing from the standard environment setup.
**Solution:**
```bash
pip install pytorch-kinematics
```

### `ImportError: libtorch_cuda_cu.so: cannot open shared object file` (PyTorch3D)
This error typically occurs when `pytorch3d` was compiled with a different CUDA version than the one currently installed or used by PyTorch.
**Symptoms:**
- Error appears when importing `pytorch3d` or running scripts that use it.
- Mismatch between `torch.version.cuda` and the system/conda `nvcc --version`.

**Solution:**
1. Ensure your `nvcc` version matches your PyTorch CUDA version. You may need to install the CUDA toolkit in your conda environment:
   ```bash
   # Example for CUDA 11.8 (if using PyTorch with CUDA 11.8)
   conda install -c "nvidia/label/cuda-11.8.0" cuda-toolkit
   ```
2. Reinstall `pytorch3d` from source:
   ```bash
   pip uninstall pytorch3d
   pip install "git+https://github.com/facebookresearch/pytorch3d.git@stable"
   ```

### `ModuleNotFoundError: No module named 'flask'`
**Solution:**
This error was observed in `models/model/evaluator.py`. It is likely due to an unused import statement.
- Remove the line `from flask.cli import F` from `models/model/evaluator.py`.
- Flask is generally not required for the core training/sampling scripts.

## Runtime Issues

### `AssertionError: Can't find provided ckpt.`
**Cause:**
The `scripts/sample.sh` (or `sample.py`) script attempts to load a model checkpoint from a path specified in `configs/sample.yaml`. The default paths in the repository may point to the author's local directories.

**Solution:**
1. Download the checkpoints from the link provided in the `README.md`.
2. Update `configs/sample.yaml` (or the relevant config file) to point to your local checkpoint paths:
   ```yaml
   sampler_bps_ckpt_pth: /path/to/your/ckpts/bps_sampler/model_200.pth
   evaluator_ckpt_pth: /path/to/your/ckpts/evaluator/model_20.pth
   ```

### Hydra Configuration Errors (e.g., `Could not find 'model/null'`)
**Cause:**
Running `scripts/sample.sh` without arguments might default to an invalid configuration if the script expects a model type.
**Solution:**
Run the script with an explicit argument, or ensure the script has a default value set:
```bash
bash scripts/sample.sh bps
```
bash scripts/sample.sh bps
```
(Note: The script has been updated to default to `bps` if no argument is provided.)

### `FileNotFoundError: .../allegro_hand_description_right.urdf`
**Cause:**
The codebase expects a specific directory structure for URDFs in `data/urdf` which might not exist by default.
**Solution:**
Create the directory and symlink the assets from `envs/assets`:
```bash
mkdir -p data/urdf/allegro_hand_description
ln -s $(pwd)/envs/assets/movable_hand_urdf/allegro/robots/allegro_hand_description_right.urdf data/urdf/allegro_hand_description/
ln -s $(pwd)/envs/assets/movable_hand_urdf/allegro/meshes data/urdf/allegro_hand_description/
```

### `AttributeError: module 'charset_normalizer' has no attribute 'md__mypyc'`
**Cause:**
Version conflict in `charset-normalizer`.
**Solution:**
Force reinstall a compatible version:
```bash
pip install --force-reinstall charset-normalizer==3.3.2
```

### `ImportError: libpython3.8.so.1.0: cannot open shared object file`
**Cause:**
Isaac Gym bindings cannot locate the Python shared library in the conda environment.
**Solution:**
Export `LD_LIBRARY_PATH` before running the script:
```bash
export LD_LIBRARY_PATH=$CONDA_PREFIX/lib:$LD_LIBRARY_PATH
```

### `AttributeError: module 'numpy' has no attribute 'float'`
**Cause:**
`np.float` was deprecated in NumPy 1.20 and removed in 1.24. Chaos ensues in legacy code.
**Solution:**
Edit `isaacgym/python/isaacgym/torch_utils.py` (line ~135) and change `dtype=np.float` to `dtype=float`.

### `ModuleNotFoundError: No module named 'ipdb'`
**Solution:**
```bash
pip install ipdb
```
