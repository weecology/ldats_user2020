# 1

Hi! 
My name is Juniper Simonis, I use they/them pronouns, and I'm going to be speaking on the LDATS R package.
This work was done in collaboration with a team of researchers based out of the Weecology lab at the University of Florida.

# 2

Our motivation is developing analytical tools to study multi-dimensional ecological time series.
Specifically, the project was started to analyze dynamics of the Portal Project rodent community, a group of 20 species of rodents 

# 3

that have been sampled monthly for over 40 years.
This time series shows each species' counts individually over time, and makes evident the changes in temporal patterns over time.

# 4

Just eye-balling these four lines, we can see periods in the time series where the species-level and the community-level dynamics change.
For example, the advent of large-magnitude cycles in the most recent time period.


# 5

A main goal of our work is then to associate changes in ecological dynamics to external stressors,
like climate change, invasive species, and human-mediated landscape alteration.
For example, this plot shows the five-year precipitation and temperature averages at the Portal site since 1980, indicating clearly that the site is experiencing warmer, drier weather.

# 6

And in evaluating the impact of these stressors on dynamics, ecological research indicates that we need to explore the potential contributions of
stochasticity, autocorrelation, cyclicity, gradual changes, and abrupt changes (known as "regime shifts").

# 7

This work was instigated by Erica Christensen as part of her disseration at UF, and she and co-authors established the methods underlying the LDATS package.

# 8

Although in that paper, the name "LDATS" doesn't appear...
it was coined later to mean Latent Dirichlet Allocation coupled with Time Series analyses

# 9

Although with the on-going expansion of the methods, we are currently considering generalizing the name to Linguistic Decomposition Algorithms coupled with Time Series analyses

# 10

In turning Christensen et al into the LDATS pacakge, we were particularly interested in standardizing, generalizing, and optimizing the methods
and encapsulating them in a simple "lm-style" top-level API
with the idea being that we keep all of the horses "under the hood", so-to-speak.

# 11

And indeed we have developed LDATS as a package (presently on CRAN in v0.2.7) that provides the user with a simple top-level API.
We are engaged in on-going work to expand the method, but the package allows the user to conduct analyses akin to Christensen et al.

# 12

To dive a bit more into the statistical underpinnings, LDATS uses a two-stage approach
with the first being focused on reducing the dimensionality of the data and the second being then analyzing the reduced dimension data

# 13

As you might imagine, given the name of the package, the initial dimension reduction technique used has been Latent Dirichlet Allocation, which is shown here in plate notation on the right

and derives its terminology from the original application of LDA to text corpora.

where there are M total documents with N total words
each observed word has term identity w and topic identity z
the model is governed by 
α Dirichlet parameter for θ, the topics-in-documents distribution and
β is the terms-in-topics distribution
This model allows individual terms to belong to multiple topics.


# 14 

In essence, however, LDA acts to decompose a document-word matrix into document-topic and topic-word matrices

# 15

In the ecological parlance... decompose a sample-species matrix into sample-guild and guild-species matrices

# 16

And the goal of using LDA here is to reduce dimensionality and thus analyze the sample-guild matrix

# 17

As the rodent data highlighted before, raw ecological time series can display a range of possible dynamics.
This is true after decomposition as well, as shown here on the rodent data, where only the last period displays intense seasonal cycling
Thus, in analyzing these time series, we need multivariate responses and flexible predictors.

# 18

Focusing first on the response models, rather than re-invent any wheels, we leverage existing multivariate regression tools.
For the initial LDATS model, we use softmax-based regression via the nnet package

# 19

However, we have hit some limitations with the softmax and so are now incoporating additional response models
Presently, we are focused on simplex-based methods from the compositions package
These are quite promising and will be included in the v0.3.0 release.

# 20

Turning our focus now to the predictor components, following Christensen et al, the LDATS model uses Bayesian change point regression, based on an indicator approach
As you can see here, in a figure from Ruggieri's paper describing the method applied to NOAA temperature data, it can identify qualitative changes in the data in the face of additional dynamics.

# 21

This is important in the context of our ecological data, as well,
where we expect there to potentially be 0, 1, or more change points with potentially variable dynamics between them, which are governed by regressors

# 22

Following established methods, we take a sequential approach to fitting the time series data where
we first estimate the change point locations unconditional on the regressors.
This provides estimates of the regressors, but they are conditional on the change points.
Thus, we then de-condition the regressors given the probabilistic change point estimates.

# 23

We can then fit the decomposed rodent data and the model selects 4 change points with varying cycle strength, as seen in the data.

# 24

One major challenge in fitting this model is handling sharp edges in the likelihood surface, which can cause optimization routines to get stuck in suboptimal parameter space.
To address this, we use parallel tempering MCMC over top of the multinomial regressions within each "segment"
For example, looking at this 2-dimensional parameter space, search trajectories could get stuck in high-density (grey) areas.
ptMCMC is able to explore the space by adding auxillary temperature chains, like shown here, that can swap information with the focal chain, allowing it to get "unstuck".

# 25 

We've coded the application of ptMCMC for LDATS within the package itself to make it easy for folks "driving the truck".

# 26 

With the idea being that within the LDATS package, the user can simply run the top-level `LDA_TS` function.
Notably, users can run, compare, and select from multiple models from a single call (using, for example, the "formulas" and "nchangepoints" arguments, which are fully factored).
All of the details of the algorithm optimizing, etc. are encompassed within the `control` list to keep the argument list tidy but generalizable.

# 27

Just a quick note here, for v0.3.0, there will be a single change to the API (seeds becomes replicates), but otherwise all other changes (which are intensive), do not require top-level API alteration.

# 28

Accompanying the package, we have 3 important vignettes that show a full "soup-to-nuts" application, compare the package to Christensen et al's results, and walk through the full code pipeline.
These vignettes and other documentation are available on the package website, weecology.github.io/LDATS.

# 29

We obviously do not have time to delve too deeply, but I did want to highlight some important functionality and ancillaries within the package. 
For example, the LDA_TS function computes summaries internally and provides them out as list components,
like the MCMC summary table for change point locations

# 30

and the MCMC summary table for regressors between the change points.

# 31

Which makes evident how many parameters can be generated even from a simple model (5 topics, 1 change point, annual cycling)!

# 32

Similarly, LDATS has nice top-level plotting functionality via `plot`

# 33

that reproduce the Christensen et al-style figures, showing the decomposition on the left and the analysis of the decomposed data on the right.

# 34

What I just showed here is what is available in v0.2.7 of LDATS, which is presently available on CRAN
It's got nice testing coverage with CI through travis and is archived on Zenodo
v0.3.0 has substantial expansions, but is still in development on GitHub

# 35

v0.3.0 includes expansion to the simulation functionality, predict methods, and a wider range of model evaluation and selection tools such as hold-outs and cross-validation
So stay tuned!

# 36

One project that is helping drive the expansion of LDATS is MATSS - Macro-ecological Analyses of Time Series Structure, which is leveraging a compendium of time series to evaluate the presence of dynamics like regime shifts in ecosystems

# 37

The MATSS package is allowing application of LDATS broadly, which is providing some key insight into ecological communities and helping expand the LDATS package.

# 38

If you are interested in getting involved with LDATS, please reach out!
The Christensen et al paper is a great starting place to read about the context, and the package documentation provides info on the computational sides of things.

# 39

And with that, I'd like to thank my co-authors and our funding sources.
Cheers!