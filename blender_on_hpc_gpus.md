# Blender on UCR GPU

## The render setup:

I found it easier to set the `.vdb` file path to the one on hpcc. When you save your `.blend` job it saves all the settings. Then transfer this `.blend` file to hpcc. Do make sure you transfer the volume data (i.e. `.vdb` file to the hpcc too).

## Download blender on hpcc :

Download the software on your home directory on HPC :

```wget https://mirror.clarkson.edu/blender/release/Blender3.1/blender-3.1.0-linux-x64.tar.xz```

Now untar the file:

`tar xvf blender-3.1.0-linux-x64.tar.xz`

and then you'll have the binary file ready to use.


## Request GPU nodes

You need to request one or few GPU nodes on HPC first. It can be submitted interactively or through sbatch joob submision.

### Interactive :
This is good only for long annimation rendering because UCR's hpc ssh session gets killed after few hours of inacitivity. So, I recommend you trying the next option.

1. Login with`ssh your_user_name@csecure.hpcc.ucr.edu`
2. Ask for GPU resources :

``sbatch --begin=2022-03-22T09:30:00 --time=24:00:00 -p gpu --gres=gpu:1 --mem=100g --cpus-per-task=4 --wrap='echo ${CUDA_VISIBLE_DEVICES} > ~/.CUDA_VISIBLE_DEVICES; sleep infinity'
``

2. check which gpu node you've got :

`squeue -u $USER`

ssh to that gpu node:

`ssh gpu-#`

3. run blender

`./blender -b ../TNG300_highres_cut6.blend -s 86 -a -- --cycles-device CUDA`

[Blender command line doc](https://docs.blender.org/manual/en/latest/advanced/command_line/render.html)

It is being rendered very fast!

### Using sbatch :

Write a submit file like this (`submit.sh`):


```
#!/bin/bash
#SBATCH --partition=gpu
#SBATCH --gres=gpu:1
#SBATCH --wrap='echo ${CUDA_VISIBLE_DEVICES} > ~/.CUDA_VISIBLE_DEVICES; sleep infinity'
#SBATCH --job-name=blgpu
#SBATCH --ntasks=1
#SBATCH --cpus-per-task=4
#SBATCH --mem=200GB

/rhome/mqezl001/vis3D/blender-3.1.0-linux-x64/blender -b /rhome/mqezl001/vis3D/TNG300_highres_cut6.blend -s 86 -a -- --cycles-device CUDA
```
Then submit your job with :
```
sbatch submit.sh
```

It will run till the render is done.


Note: If your rendering is really slow use multiple GPUs, Belnder scales over multiple GPUs. But keep in mind that GPUs are scarce resources, so other people may need it too.


## Check memory and GPU usage:

The `top` command does not see GPUs, you need other softwares. The native software for NVIDIA GPUs we have on hpcc is `nvidia-smi`, but we do not have it installed. So, I found [`gpustat`](https://github.com/wookayin/gpustat) python package based on the NVIDIA.  You need exactly `python3.9` for that make a new conda environment and then install it : `pip install gpustat`.

After your job started login to the corresponding gpu node and type `gpustat --watch` it shows a dynamic log of the gpu usage on that node. You should be at least using one of the GPUs (look at figure below). If you don't see your user name  there, your code is not being run on GPU and you need to go back and check your job submission. The common pitfall is forgetting the `--` after `-a` in the submit file above.

![](https://i.imgur.com/9OTcXdK.png)
