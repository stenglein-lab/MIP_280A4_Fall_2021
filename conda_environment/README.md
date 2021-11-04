This directory contains files necessary to create a conda environment that includes the tools necessary to complete the exercises in this course.


To create a new conda environment, change to the directory containing the mip_280_conda.yaml file and run:
```
conda env create --prefix=$HOME/bio_tools --file mip_280_conda.yaml
```

To **activate** this conda environment (necessary to run tools in the environment), run: 
```
conda activate $HOME/bio_tools 
```

To **update** this conda environment with additional conda packages, add them to the .yaml file, change to the directory containing the updated yaml file, and run:
```
conda env update --prefix=$HOME/bio_tools --file mip_280_conda.yaml
```

