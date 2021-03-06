# Myokit & Pints examples

This repository contains examples showing how to fit [Myokit](https://github.com/MichaelClerx/myokit/) models to data using the [Pints](https://github.com/pints-team/pints) optimisation & inference framework.

Pints comes with [examples](https://github.com/pints-team/pints/blob/master/examples/README.md) and [API documentation](https://pints.readthedocs.io/). [This pre-print](https://arxiv.org/abs/1812.07388) describes Pints' design, information on citing it is given [here](https://github.com/pints-team/pints/blob/master/CITATION).

Similarly, Myokit comes with some [example models](http://myokit.org/examples/) and [API docs](https://myokit.readthedocs.io). [This publication in PBMB](https://doi.org/10.1016/j.pbiomolbio.2015.12.008) describes Myokit, and can be cited using the information [here](https://github.com/MichaelClerx/myokit/blob/master/CITATION).

Please see the [Myokit](https://github.com/MichaelClerx/myokit/) and [Pints](https://github.com/pints-team/pints) repos for installation instructions for both tools.

## General approach

- [Models are written in Myokit's MMT syntax](https://myokit.readthedocs.io/syntax/index.html), usually by [downloading a CellML model](https://models.cellml.org/electrophysiology) and then [importing it](https://myokit.readthedocs.io/api_formats/cellml.html).
- [Simulations are run](https://myokit.readthedocs.io/api_simulations/Simulation.html) using the `Simulation` class, which uses CVODE to solve the ODEs.
  - For ion-channel stuff with piecewise constant voltage-step protocols (no ramps or sine waves) it's usually faster to use Myokit's [Hodgkin-Huxley model](https://myokit.readthedocs.io/api_library/hh.html) or [Markov model](https://myokit.readthedocs.io/api_library/markov.html) classes to run analytical simulations)
- [A pints.ForwardModel](https://nbviewer.jupyter.org/github/pints-team/pints/blob/master/examples/writing-a-model.ipynb) is wrapped around a Myokit simulation.
- [A pints.ErrorMeasure](https://pints.readthedocs.io/en/latest/error_measures.html) or [pints.LogLikelihood](https://pints.readthedocs.io/en/latest/log_likelihoods.html) is defined
- [Optimisation](https://nbviewer.jupyter.org/github/pints-team/pints/blob/master/examples/optimisation-first-example.ipynb) or [Bayesian inference](https://nbviewer.jupyter.org/github/pints-team/pints/blob/master/examples/sampling-first-example.ipynb) is run.

## Examples: Simulation with Myokit

1. [Running whole-cell simulations in Myokit](https://myokit.readthedocs.io/guide/simulations.html)
2. [Running ion current simulations](https://nbviewer.jupyter.org/github/pints-team/myokit-pints-examples/blob/master/ion-currents/ion-current-models.ipynb)
3. [Exact solvers for voltage-step protocols](https://nbviewer.jupyter.org/github/pints-team/myokit-pints-examples/blob/master/ion-currents/exact-simulation-of-step-protocols.ipynb)
4. [Simulating a novel protocol with steps and ramps](https://nbviewer.jupyter.org/github/pints-team/myokit-pints-examples/blob/master/ion-currents/steps-and-ramps.ipynb)
5. [Simulating a novel protocol with steps and a sine wave](https://nbviewer.jupyter.org/github/pints-team/myokit-pints-examples/blob/master/ion-currents/steps-and-sine-waves.ipynb)
6. [Simulating arbitrary points with 'data clamp'](https://nbviewer.jupyter.org/github/pints-team/myokit-pints-examples/blob/master/ion-currents/data-clamp.ipynb)

## Examples: Inference using Myokit and PINTS

1. [Fitting conductances in a whole-cell model](https://github.com/MichaelClerx/myokit-pints-examples/tree/master/whole-cell-conductances)
2. [A real-life example](https://github.com/CardiacModelling/FourWaysOfFitting/blob/master/python/model.py) of a ForwardModel class used for sine-wave siulations as well as analytical step-protocol simulations
3. An example of a [parameter transform](https://github.com/CardiacModelling/FourWaysOfFitting/blob/master/python/transformation.py). This could be used in a forward model implementation, so that the parameters the model presents to the optimiser/sampler are in a transformed space.
4. [Example code](https://pints.readthedocs.io/en/latest/boundaries.html) for a [pints.Boundaries](https://pints.readthedocs.io/en/latest/boundaries.html) object for a hERG model. This could be adapted to be a [pints.LogPrior](https://pints.readthedocs.io/en/latest/log_priors.html).

## Some recommendations for fitting

1. Whenever possible, fit to **time-series data**, not to processed values such as time constants, IV-, or (in)activation curves. [This figure](https://www.biorxiv.org/content/10.1101/609875v1.full#F11) shows why: A score function defined on processed values (method 2 in the paper) can have a complex surface, full of local minima, while similar functions defined on the underlying time-series data (methods 3 and 4 in the paper) are convex and smooth.
2. Define prior expectations on transition rates as well as parameters. See [this figure](https://www.biorxiv.org/content/10.1101/609875v1.full#F3) and the [supplementary materials](https://www.biorxiv.org/content/10.1101/609875v1.supplementary-material).
3. Searching in a log-transformed parameter space [can make your problem more convex](https://dx.doi.org/10.1093/bioinformatics/btz020) (which is a good thing).
4. Use an analytic solver if possible, if using an adaptive ODE solver make sure you [set very fine tolerances](https://mirams.wordpress.com/2018/10/17/ode-errors-and-optimisation/). In Myokit, this can be done with [`Simulation.set_tolerance()`](https://myokit.readthedocs.io/api_simulations/Simulation.html#myokit.Simulation.set_tolerance).
5. Test the reliability of your fit by running repeated fits from different starting points (e.g. sampled uniformly from your prior).
6. Before doing any experiments, test the whole set-up with simulated data (and simulated noise).
