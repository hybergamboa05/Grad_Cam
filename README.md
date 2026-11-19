# Grad-CAM Workshop: See What Your Model Is Looking At

In this workshop you'll build a **Grad-CAM heatmap** from scratch in PyTorch, check it against the `pytorch-grad-cam` library, and run a sanity test that proves the heatmap actually means something.

- **No GPU needed.** CPU is fine.
- **No dataset to download.** The photos ship with scikit-image, and the ResNet-18 weights download automatically the first time you run Step 2 (about 45 MB).

---

## Option A: Google Colab (easiest, no setup)

1. Upload the notebook to [colab.research.google.com](https://colab.research.google.com) (File → Upload notebook).
2. Run the cells top to bottom. The `%pip install` cell in Step 0 adds the Grad-CAM library.
3. Done. PyTorch, torchvision, and scikit-image are already installed in Colab.

If you'd rather run it on your own machine, follow Option B.

---

## Option B: Run locally

### What you need

- **Python 3.10, 3.11, or 3.12** (3.12 recommended). Python 3.13+ may not have wheels for every package yet.
- About **2 GB** of free disk space (PyTorch is big).
- An internet connection for the first run.

Check your Python version first:

| Mac | Windows |
|---|---|
| `python3 --version` | `python --version` |

If it's missing or too old, install it from [python.org/downloads](https://www.python.org/downloads/). **On Windows, tick "Add python.exe to PATH" on the first installer screen.** It's easy to miss and causes most of the problems below.


---

### macOS setup

Open **Terminal** (Cmd + Space, type "Terminal") and run:

```bash
# 1. Go to the workshop folder
cd path/to/workshop-folder

# 2. Create a virtual environment named "venv"
python3 -m venv venv

# 3. Activate it (you'll see (venv) at the start of your prompt)
source venv/bin/activate

# 4. Upgrade pip, then install everything
pip install --upgrade pip
pip install -r requirements.txt

# 5. Register this environment as a Jupyter kernel
python -m ipykernel install --user --name gradcam-workshop --display-name "Python (gradcam-workshop)"

# 6. Launch Jupyter
jupyter notebook
```

### Windows setup

Open **PowerShell** (Start menu → type "PowerShell") and run:

```powershell
# 1. Go to the workshop folder
cd path\to\workshop-folder

# 2. Create a virtual environment named "venv"
python -m venv venv

# 3. Activate it (you'll see (venv) at the start of your prompt)
venv\Scripts\Activate.ps1

# 4. Upgrade pip, then install everything
python -m pip install --upgrade pip
pip install -r requirements.txt

# 5. Register this environment as a Jupyter kernel
python -m ipykernel install --user --name gradcam-workshop --display-name "Python (gradcam-workshop)"

# 6. Launch Jupyter
jupyter notebook
```

Using **Command Prompt** instead of PowerShell? Step 3 becomes `venv\Scripts\activate.bat`.

If step 3 fails with a red "running scripts is disabled" error, see [Troubleshooting](#troubleshooting).

---

## Running the notebook

1. In the Jupyter tab that opens, click the workshop notebook (`.ipynb`).
2. Go to **Kernel → Change kernel** (or **Select Kernel** in VS Code) and pick **Python (gradcam-workshop)**.
3. Run cells from **top to bottom** with `Shift + Enter`.
4. Compare your output to each **CHECK** note. If something doesn't match, ask a helper before moving on.

### Quick environment test

Run this in a notebook cell. If it prints a version and `OK`, you're set:

```python
import torch, torchvision, skimage, pytorch_grad_cam
print("PyTorch:", torch.__version__)
print("torchvision:", torchvision.__version__)
print("OK")
```

### Using VS Code instead of Jupyter?

Install the **Python** and **Jupyter** extensions, open the notebook, click **Select Kernel** (top right), and choose the `venv` / `gradcam-workshop` environment.

---

## Troubleshooting

### Python and pip problems

**`python: command not found` (Mac)**
Use `python3` instead of `python` outside the virtual environment. Once `(venv)` is active, `python` works.

**`'python' is not recognized as an internal or external command` (Windows)**
Python isn't on your PATH. Reinstall it and tick **"Add python.exe to PATH"**, or try `py -m venv venv` (the Windows launcher) in place of `python -m venv venv`. Open a *new* terminal afterward.

**Windows opens the Microsoft Store when you type `python`**
Windows is using its placeholder. Install Python from python.org, then turn off the placeholder: Settings → Apps → Advanced app settings → App execution aliases → switch off the two "python" entries.

**`pip: command not found` / `pip is not recognized`**
Use `python -m pip install ...` (Windows) or `python3 -m pip install ...` (Mac). It always works if Python does.

**`No matching distribution found for torch`**
Your Python version is probably too new (3.13+) or you're on 32-bit Python. Install Python 3.12 (64-bit), recreate the venv, and try again.

**`error: externally-managed-environment` (Mac with Homebrew Python)**
You're installing outside a virtual environment. Run `source venv/bin/activate` first and make sure `(venv)` shows in your prompt.

**Install is very slow or times out**
PyTorch is a large download. Try again on a faster connection, or use `pip install --default-timeout=300 -r requirements.txt`.

### Activation problems (Windows)

**`venv\Scripts\Activate.ps1 cannot be loaded because running scripts is disabled on this system`**
PowerShell's execution policy is blocking it. Run this once, then activate again:

```powershell
Set-ExecutionPolicy -Scope CurrentUser -ExecutionPolicy RemoteSigned
```

Or just use Command Prompt with `venv\Scripts\activate.bat`.

**Mac: `(venv)` doesn't appear after activating**
Make sure you ran `source venv/bin/activate` (with `source`) from the folder that contains `venv`.

### Notebook and kernel problems

**`ModuleNotFoundError: No module named 'torch'` (or `skimage`, `pytorch_grad_cam`) even though you installed it**
Jupyter is using the wrong Python. Switch the kernel to **Python (gradcam-workshop)** (Kernel → Change kernel). If it isn't in the list, redo the `ipykernel install` step with your environment activated. As a last resort, run `%pip install -r requirements.txt` in a notebook cell, then **restart the kernel**.

**`ModuleNotFoundError: No module named 'skimage'`**
The package name differs from the import name. Install it with `pip install scikit-image`.

**`ModuleNotFoundError: No module named 'pytorch_grad_cam'`**
Install it with `pip install grad-cam==1.5.7`. The pip name is `grad-cam`, but you import `pytorch_grad_cam`.

**`jupyter: command not found`**
Your environment isn't activated, or Jupyter didn't install. Activate it and run `pip install jupyter`, or launch with `python -m notebook`.

**NumPy errors like "A module that was compiled using NumPy 1.x cannot be run in NumPy 2.x"**
A version mismatch. Fix with:

```bash
pip install "numpy<2"
```

Then restart the kernel.

### Model download problems (Step 2)

**`URLError`, `SSL: CERTIFICATE_VERIFY_FAILED`, or a download timeout**
The ResNet-18 weights download on first use. Check you're online. On a school or work network, a firewall or proxy may block it, so try a phone hotspot.

On Mac with python.org Python, SSL errors can be fixed by running `/Applications/Python 3.12/Install Certificates.command` (adjust the version number).

**Corrupted or partial download**
Delete the cached weights and rerun the cell:

- Mac: `~/.cache/torch/hub/checkpoints/`
- Windows: `C:\Users\<you>\.cache\torch\hub\checkpoints\`

### Errors while running the Grad-CAM cells

**Your outputs don't match the CHECK notes**
Small differences in decimals between PyTorch versions are normal (for example 85% vs 84.7%). Big differences usually mean a cell was skipped or run out of order.

**`RuntimeError: One of the differentiated Tensors ...` or `element 0 of tensors does not require grad`**
Steps 5 and 6 depend on each other. Don't run Step 6 on its own, and don't run Step 5 inside a `torch.no_grad()` block. **Fix:** Kernel → **Restart & Run All** (stop before the "Your turn" cells).

**Heatmap looks odd after an earlier error in Step 5**
If Step 5 crashes before `hook.remove()`, the hook stays attached and later cells misbehave. Restart the kernel and run again from the top.

**`AssertionError: Set TARGET_A and TARGET_B to class names...`**
That's expected. It means you haven't filled in the TODO yet. Replace `None` with a class name in quotes, like `"tabby"`.

**`ValueError: '...' is not a class name`**
Class names must match exactly, including spelling and capitalization. Copy one directly from the `top5()` output. Don't copy the number or percentage.

**Figures don't show up**
Make sure the cell finishes with `plt.show()`. If you're using a plain Python script instead of a notebook, run `pip install pyqt5` or save with `plt.savefig("out.png")`.

### Still stuck?

1. **Restart the kernel and run all cells** from the top. This fixes most weird states.
2. Copy the *last line* of the error message and search it, or send it to a helper.
3. Include your OS, `python --version`, and the output of `pip list` when asking for help.

---

## What's in the notebook

| Step | What you do |
|---|---|
| 0–1 | Install and import |
| 2–4 | Load ResNet-18 and see its top-5 predictions |
| 5–8 | Build Grad-CAM yourself: hook, gradients, weighted sum, heatmap |
| 9 | Check your result against `pytorch-grad-cam` (similarity should be **1.000**) |
| 10 | Sanity test: cover the hottest 20% vs. coldest 20% of the image |
| 11 | Your turn: explain two objects in one image, and a wrong answer on a rocket photo |