# ipyexperiments
experiment containers for jupyter/ipython for GPU and general RAM re-use

# About

This module's main purpose is to help calibrate parameters in deep learning notebooks to fit the available GPU and General RAM, but, of course, it can be useful for any other use where memory limits is a constant issue.

Using this framework you can run multiple consequent experiments without needing to restart the kernel all the time, especially when you run out of GPU memory - the familiar to all "cuda: out of memory" error. When this happens you just go back to the notebook cell where you started the experiment, change the parameters, and re-run the updated experiment until it fits the available memory. This is much more efficient and less error-prone then constantly restarting the kernel, and re-running the whole notebook.

As an extra bonus you get access to the memory consumption data, so you can use it to automate the discovery of the parameters to suit your hardware's unique memory limits.

The idea behind this module is very simple - let's implement a function-like functionality, where its local variables get destroyed at the end of its run, giving us memory back, except it'll work across multiple jupyter notebook cells (or ipython). In addition it also run `gc.collect()` to immediately release badly behaved variables with circular references, and reclaim general and GPU RAM. The latter also happens to help discover memory leaks.

XXX: This is a prototype and it currently assumes you use `pytorch` so it doesn't require it (since currently `pytorch` is in pre-release stage), so I assume you already have it installed. This will change though and it'll become optional, other deeplearning frameworks can be deployed too.

## Usage

Here is an example with using code from the [`fastai`](https://github.com/fastai/fastai) library. I added a visual leading space to demonstrate the idea, but of course it won't be valid python.

```
cell 1: exp1 = IPyExperiments()
cell 2:   learn1 = language_model_learner(data_lm, bptt=70, drop_mult=0.3, pretrained_model=URLs.WT103)
cell 3:   learn1.lr_find()
cell 4: del exp1
cell 5: exp2 = IPyExperiments()
cell 6:   learn2 = language_model_learner(data_lm, bptt=70, drop_mult=0.3, pretrained_model=URLs.WT103)
cell 7:   learn2.lr_find()
cell 8: del exp2
```

## Demo
[demo notebook](https://github.com/stas00/ipyexperiments/blob/master/demo.ipynb)

## Installation
pip install git+https://github.com/stas00/ipyexperiments.git

## API

1. Create an experiment object:
   ```python
   exp1 = IPyExperiments()
   ```

2. Get intermediary experiment usage stats:
   ```python
   consumed, reclaimed, available = exp1.get_stats()
   ```
   3 dictionaries are returned. This way is used so that in the future new entries could be added w/o breaking the API. The memory stats are in bytes.

   ```python
   print(consumed, reclaimed, available)
   {'gen_ram': 2147500032, 'gpu_ram': 0} {'gen_ram': 0, 'gpu_ram': 0} {'gen_ram': 9921957888, 'gpu_ram': 7487881216}
   ```
   This method is useful for getting stats half-way through the experiment.

3. Finish the experiment, delete local variables, reclaim memory. Return and print the stats:
   ```python
   final_consumed, final_reclaimed, final_available = exp1.finish() # finish experiment
   print("\nNumerical data:\n", final_consumed, final_reclaimed, final_available)
   ```

   If you don't care for saving the experiment numbers, instead of calling `finish()`, you can just do:
   ```python
   del exp1
   ```
   If you re-run the experiment w/o either calling `exp1.finish()` or `del exp1`, e.g. if you decided to abort it half-way to the end, then the constructor `IPyExperiments()` will trigger a destructor first and therefore previous experiment's stats will be printed first.
