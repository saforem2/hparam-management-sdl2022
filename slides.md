---
title: 'Data Science on Polaris'
center: false
width: 960
height: 540
transition: slide
background: #1c1c1c
preloadIframes: false
highlightTheme: 'monokai'
keyboardCondition: 'focused'
maxScale: 20.0
css:
 - ./css/local.css
date created: Monday, May 16th 2022, 7:07:14 am
date modified: Wednesday, September 21st 2022, 9:18:51 pm
---

<!-- .slide bg="#1c1c1c" -->

<grid align="center" drop="center" drag="70 25" style="background-color:#282828; border-radius:8px;">

<span style="border-bottom:3px solid var(--r-header-accent); font-weight:800; font-size:1.75em; margin-bottom:0px;">Hyperparameter Management</span>

<span style="font-size:0.8em; line-height:0.8em; margin-bottom:0px; margin-top:0px;">[<i class="fab fa-github fa-1x" alt="`fas:Github`"/> argonne-lcf / SDL Workshop](https://github.com/argonne-lcf/sdl_workshop)</span> <!-- .element style="color:#00CCFF!important; font-family:'JuliaMono', monospace" -->

</grid>

<grid drag="100 10" drop="0 70" align="bottomleft" >

<a href="https://www.samforeman.me"><i class="fas fa-home fa-1x" alt="`fas:Home`" /></a> Sam Foreman <!-- .element style="color:#505050;" -->

<span style="font-size:0.9em; color:#505050; padding:0px; margin:0px; text-align:center!important;">2022-09-28
</grid>

<grid drag="100 20" drop="bottom" align="bottomright" >
<img src="https://raw.githubusercontent.com/saforem2/physicsSeminar/main/assets/Argonne_cmyk_white.svg" alt="Argonne National Laboratory">
</grid>


---

<!-- .slide template="[[template]]" style="text-align:left;" -->

# Overview

- Motivation
	- Need for experiment management:
		- `./outputs/good-model`
		- `./outputs/good-model-1`
		- `./outputs/better-model-final`
		- `...`
	- Better:
		- Timestamps?
	  ```python
	  import os
	  import datetime
	  from pathlib import Path
	  now = datetime.datetime.now()
	  tstamp = now.strftime('%Y-%m-%d-%H%M%S')
	  outdir = Path(os.getcwd()).joinpath(tstamp)
	  ```


note: MOTIVATIONS
- Motivate need for experiment management
	- Running experiments
	- Consistent locations
- CPU Only?
- `1` GPU?
- `>1` GPU?
	- Independent workers or communication?
		- How do you handle communication?
	- Do you use distributed training?
		- Specific software / framework?
		- How do you launch your application?

- How do you specify the details / configuration of the experiment?
	- Directly from the commandline? (`sys.argv`, `ArgumentParser()`, etc.)
	- Environment variables?
	- Configuration files?
	- Additional software / Libraries?
	- How do you ensure the options you specify are the ones being used?
- How do you measure model performance?
	- Manually?
	- Other software?
	- How do you keep track of changes to the experiment?
	- Aggregation tools?

---

<!-- .slide template="[[template]]" bg="#1c1c1c" -->

# Getting Started

- Start by requesting an interactive job:

- **Polaris**:
  ```
  qsub -A SDL_WORKSHOP -q "prod" \
      -l select=32 \
	  -l walltime=12:00:00 \
	  -l filesystems=eagle:home:grand \
	  -I
  ```

- **ThetaGPU**:
  ```
  qsub -A SDL_WORKSHOP -q "default" \
	  -n=1 \
	  -t=01:00 \
	  --attrs="filesystems=home,eagle,grand,theta-fs0" \
	  -I
  ```

---

<!-- .slide template="[[template]]" bg="#1c1c1c" style="text-align:center!important;"-->

<img src="https://hydra.cc/img/logo.svg" width="10%" display="inline-block" style="margin-top:-8px; padding-right:5px;" align="center" ><a href="https://hydra.cc"><span style="line-height:2em; vertical-align:baseline; font-size:2.0em; font-weight:700; border-bottom: 2px solid #7AA5BC; color:#f5f6f7;">Hydra</span></a>

<span style="color:#7AA5BC; text-align:right;"> _A framework for elegantly configuring complex applications_</span>

<grid drop="0 60" drag="100 20" align="left">

<split even>

**No boilerplate** <!-- .element class="standout" style="background:#7AA5BC; color:#303030;" -->
**Powerful Configuration** <!-- .element class="standout" style="background:#7AA5BC; color:#303030;" -->
**Pluggable Architecture** <!-- .element class="standout" style="background:#7AA5BC; color:#303030;" -->
</split>
</grid>


---

<!-- .slide template="[[template]]" bg="#1c1c1c" -->

<img src="https://hydra.cc/img/logo.svg" width="15%" display="inline-block" style="margin-top:-8px; padding-right:5px;" align="center" ><a href="https://hydra.cc"><span style="line-height:2.5em; vertical-align:baseline; padding-top:20px; font-size:2.0em; font-weight:700; border-bottom: 2px solid #7AA5BC; color:#EFEFEF;">Hydra</span></a>
- Key Features:
	- Hierarchical configuration composable from multiple sources
	- Configuration can be specified **or overridden** from the command line
	- Dynamic command line tab completion
- Used for:
	- Experiment configuration
	- Experiment execution
	- Run locally or launch remotely
	- `multi-run`: Run multiple jobs with different arguments with a single command

---

<!-- .slide template="[[template]]" bg="#1c1c1c" -->

# Quick Start

- We will cover a simple example demonstrating the basic functionality
	- There's a _whole lot more_ to Hydra; check out their [tutorial](https://hydra.cc/docs/tutorials/basic/your_first_app/simple_cli/)
- To install:
  ```bash
  python3 -m pip install --upgrade "hydra-core" "hydra_colorlog"
  ```

---

<!-- .slide template="[[template]]" bg="#1c1c1c" -->

# Simple Example

- We include below a simple example that simply prints the configuration it receives.
  ```python
  import hydra  
  from omegaconf import DictConfig, OmegaConf  

  @hydra.main(version_base=None)  
  def main(cfg: DictConfig) -> None:  
	  print(OmegaConf.to_yaml(cfg))  
  
  if __name__ == "__main__":  
	  main()
  ```

- You can add config values via the command line (the `+` indicates that the field is new)
  ```shell
  $ python my_app.py +network.hidden_size=64 +data.batch_size=512
  ```
  ```
  network:
	  hidden_size: 64
  data:
	  batch_size: 512
  ```

---

<!-- .slide template="[[template]]" bg="#1c1c1c" -->

## Using Configs

- ⚙️ `./conf/config.yaml`:
  ```
  network:
	  hidden_size: 200
	  activation_fn: relu
	  dropout_rate: 0.25
  ```

- 🐍 `./main.py`:
  ```
  import hydra
  from omegaconf import DictConfig, OmegaConf

  @hydra.main(version_base=None, config_path='conf', config_name='config')
  def main(cfg: DictConfig) -> None:
	  print(OmegaConf.to_yaml(cfg))

  if __name__ == '__main__':
	main()
  ```


---

<!-- .slide template="[[template]]" align:left!important;" -->

![WandB|450](https://raw.githubusercontent.com/wandb/wandb/master/.github/wb-logo-darkbg.png) <!-- .element align="top" -->

_W&amp;B is the machine learning platform for developers to build better models faster_

- [Experiment tracking](https://docs.wandb.ai/guides/track): Visualize experiments in real time
- [Hyperparameter Tuning](https://docs.wandb.ai/guides/sweeps): Optimize models quickly
- [Data and Model Versioning](https://docs.wandb.ai/guides/data-and-model-versioning): Version datasets and models
- [Model Management](https://docs.wandb.ai/guides/models): Manage the model lifecycle from training to production
- [Data Visualization](https://docs.wandb.ai/guides/data-vis): Visualize predictions across model versions
- [Collaborative Reports](https://docs.wandb.ai/guides/reports): Describe and share findings with colleagues
- [Integrations](https://docs.wandb.ai/guides/integrations): `PyTorch`, `Keras`, 🤗 `HuggingFace`, and more!

---

<!-- .slide template="[[template]]" align:left!important;" -->

<grid drag="20 25" drop="bottomleft">

![WandB|250](https://raw.githubusercontent.com/wandb/wandb/master/.github/wb-logo-darkbg.png) <!-- .element align="topleft" -->
</grid>

# Quick Start

1. Install and login
  ```
  $ python3 -m pip install wandb
  $ wandb login
  ```

2. Start a new run
  ```python
  wandb.init(project='my-project')
  ```

3. Track metrics
  ```python
  wandb.log({'accuracy': train_acc, 'loss': train_loss})
  ```

---

<!-- .slide template="[[template]]" align:left!important;" -->

<grid drag="20 15" drop="topleft">

![WandB](https://raw.githubusercontent.com/wandb/wandb/master/.github/wb-logo-darkbg.png) <!-- .element align="topleft" -->
</grid>

<grid drag="100 70" drop="center" flow="row">

<img src="./assets/wandb-batch-loss.svg" align="left" width=40%>
<img src="./assets/wandb-batch-acc.svg" align="right" width=40%>
</grid>

---

<!-- .slide template="[[template]]" align:left!important;" -->

<grid drag="20 15" drop="topleft">

![WandB](https://raw.githubusercontent.com/wandb/wandb/master/.github/wb-logo-darkbg.png) <!-- .element align="topleft" -->
</grid>

<img src="./assets/wbsweep.svg" width=90%>

---

<!-- .slide bg="#1c1c1c" align:left!important;" style="height:100%!important;" -->

<iframe width=100% height=100% data-src="https://wandb.ai/alcf-mlops/sdl_workshop-hyperparameterManagement_src_hplib/reports/Sweep-Report--VmlldzoyNzQ0ODEz" data-background-interactive style="border:none;"></section>

---

<style>

:root {
	--cm-inline-background: #242424;
	--cm-inline-foreground: #00CCFF;
    --r-heading-text-transform: none;
    --r-heading-font: 'Inter', 'Arial', "OpenSans-Bold", "Open Sans", Helvetica, Impact, sans-serif;
    --r-main-background-color: #1c1c1c!important;
    --r-main-font: 'Inter', "Arial", "Open Sans", "Coming Soon", "SourceSansPro", Helvetica Neue, sans-serif;
    --r-heading-letter-spacing: -0.45px;
    --r-heading-word-spacing: 0.5px;
    --r-heading-text-transform: none;
    --r-heading-text-shadow: none;
    --r-heading-font-weight: 700;
    --r-heading1-text-shadow: none;
    --r-main-font-size: 22px;
	--r-main-line-height: 1.5em;
    --r-monospace-font-size: 18px;
    --r-heading1-size: 1.33em;
    --r-heading2-size: 1.25em;
    --r-heading3-size: 1.2em;
    --r-heading4-size: 1.15em;
    --r-heading5-size: 1.05em;
    --r-heading6-size: 1.025em;
	--r-heading-line-height:1.5em;
    --r-code-font: 'JuliaMono', 'Hack', 'VictorMono', "agave Nerd Font", monospace;
    --r-link-color: #03A9F4;
    --r-link-color-dark: #f92672;
    --r-link-color-hover: #63ff51;
	--r-accent-color: #77CA29;
    --r-controls-color: #228BE6;
    --r-progress-color: #404040;
	--r-header-accent: #1E8BC9;
    --r-selection-background-color: rgba(255, 255, 0, 0.15);
    --r-selection-color: rgb(255, 255, 0);
    --r-main-color: #c8c8c8;
    --text-muted: #757575;
    --text-faint: #404040;
    --r-heading-color: #FFF;
    --r-background-color: #ffffff;
	--cm-keyword: #c792ea;
	--cm-atom: #f78c6c;
	--cm-number: #ff5370;
	--cm-type: #decb6b;
	--cm-def: #82aaff;
	--cm-property: #c792ea;
	--cm-variable: #f07178;
	--cm-variable-2: #EEFFFF;
	--cm-variable-3: #f07178;
	--cm-definition: #82aaff;
	--cm-callee: #89ddff;
	--cm-qualifier: #decb6b;
	--cm-operator: #89ddff;
	--cm-hr: #98e342;
	--cm-link: #696d70;
	--cm-error-bg: #ff5370;
	--cm-header: #da7dae;
	--cm-builtin: #ffcb6b;
	--cm-meta: #ffcb6b;
	--cm-matching-bracket: #FFFFFF;
	--cm-tag: #ff5370;
	--cm-tag-in-comment: #ff5370;
	--cm-string-2: #f07178;
	--cm-bracket: #ff5370;
	--cm-comment: #676e95;
	--cm-string: #c3e88d;
	--cm-attribute: #c792ea;
	--cm-attribute-in-comment: #c792ea;
	--cm-background-color: #202020;
	--cm-active-line-background-color: #353a50;
	--cm-foreground-color: #bdbdbd;
      -webkit-font-smoothing:subpixel-antialiased;
	--chart-color-1: #ff00ff;
	--chart-color-x: rgb(255,255,255);
}

.standout{
	background: var(--cm-background-color);
	padding:5px;
	font-weight:700;
	border-radius:6px;
}

.reveal pre {
  display:block;
  margin:auto;
  width:auto;
  font-family: var(--r-code-font);
  font-size: var(--r-monospace-font-size);
  padding: auto;
  white-space: pre-wrap;
}

.reveal p {
  margin:auto!important;
  padding:auto!important;
}

.reveal pre code {
    display: inline-block;
	top: 2px;
	white-space: pre;
    bottom: 2px;
	margin:auto;
	padding:auto;
    font-size: 0.8em;
    background:var(--cm-background-color);
    color: var(--cm-foreground-color)!important;
	text-align: justify;
    letter-spacing: -0.45px!important;
    word-spacing: -0.5px!important;
}

.reveal {
    font-family: var(--r-main-font), sans-serif;
    font-size: var(--r-main-font-size);
    font-weight: normal;
    color: var(--r-main-color);
    background-color: var(--r-main-background-color);
}


.reveal blockquote p {
  color: var(--text-muted);
  font-style: normal !important;
  font-align: left;
  display: inline;
  text-align: left;
  min-width:max-content;
  max-width:min-content;
}

.reveal blockquote em{
  color: var(--text-muted);
  text-align: left;
}

.reveal blockquote {
  border-radius: 8px !important;
  margin: 0.5rem 0rem 0.5rem 0rem;
  text-align: left;
  padding-top: 1rem;
  padding-left: 2rem;
  padding-bottom: 1rem;
  padding-right: 2rem;
  width: auto;
  font-style: normal !important;
}

.reveal blockquote {
	font-size: unset;
	margin: auto;
	padding:auto;
}


.reveal ul, ol {
	text-align:left;
}

.reveal ul ul,
.reveal ul ol,
.reveal ol ol,
.reveal ol ul {
  text-align:left;
}

.reveal ul ul {
    list-style: circle;
}
.container {
  position: relative;
}

.make-it-pop {
  filter: drop-shadow(0 0 10px purple);
}

.bottomright {
  position: absolute;
  bottom: 8px;
  right: 16px;
  font-size: 18px;
}


@media (max-width: 95%) {
  section {
    -webkit-flex-direction: column;
    flex-direction: column;
  }
}

.row {
  display: flex;
}

.column {
  flex: 50%;
}

.horizontal_dotted_line{
  border-bottom: 2px dotted gray;
}

.footer {
  font-size: 60%;
  vertical-align:bottom;
  color:#bdbdbd;
  font-weight:400;
  margin-left:-5px;
}
.note {
  color:#f8f8f8;
  border-radius:8px;
  background-color:#35353540;
  border-color:#66666640;
  padding: auto;
  margin:auto;
}

.callout {
  border-left: 6px solid rgb(var(--callout-color));
  background-color: #;
  opacity:95%;
  text-align:left;
  margin-left: 4%;
  border-radius:2px;
  page-break-inside: avoid;
}

.callout > ul,ol {
	line-height:1.5em!important;
}

.callout-title {
  display: flex;
  text-align:left;
  color: #f8f8f8;
  padding-left:10px;
  gap: 10px;
  font-size:1em;
  top: 0;
  position:relative;
  margin-top:0px!important;
  margin-bottom:0em!important;
  padding-top:0px!important;
  padding-bottom:0px!important;
  border-radius:2px;
  background-color: rgba(var(--callout-color), 0.3);
}

.callout-content {
	padding: 1px!important;
	top: 0;
	margin-left: 1%;
    padding-bottom:0.75em;
	padding:auto;
    line-height:1em!important;
}

.reveal .code-wrapper code {
	width: 98%;
}
.reveal code {
  font-family: var(--r-code-font);
  text-transform: none;
  tab-size: 4;
  /*border:1px solid red;*/
  background:var(--cm-background-color);
  color: var(--cm-foreground-color);
  border-radius:2px;
  letter-spacing: -0.45px!important;
  word-spacing: -0.5px!important;
}

.reveal pre code {
	padding: auto;
	border:none;
	border-radius:2px;
	font-size:0.9em;
	margin:auto;
	background: var(--cm-background-color);
}
.reveal p code {
  font-family: var(--r-code-font);
  text-transform: none;
  tab-size: 4;
  padding:auto;
  /* border:1px solid green; */
  font-size:0.9em!important;
  line-height:inherit;
  background:var(--cm-inline-background);
  color: var(--cm-inline-foreground);
  border-radius:3px;
  letter-spacing: -0.45px!important;
  word-spacing: -0.5px!important;
}
#customcontrols > ul {
  display: none!important;
}

#customcontrols button {
  display: none!important;
}

.reveal .slides > section.present, .reveal .slides > section > section.present {
  min-height: 100% !important;
  display: flex !important;
  flex-direction: column !important;
  justify-content: center !important;
  position: absolute !important;
  top: 0 !important;
}
section > h1 {
  position: absolute !important;
  top: 0 !important;
  margin-left: auto !important;
  margin-right: auto !important;
  left: 0 !important;
  right: 0 !important;
}

h1 {
	border-bottom: 3px solid var(--r-header-accent);
	text-align: left!important;
	min-width: max-content;
	max-width: min-content;
}

.print-pdf .reveal .slides > section.present, .print-pdf .reveal .slides > section > section.present {
  min-height: 770px !important;
  position: relative !important;
}

.hljs {
    background: var(--cm-background-color) !important;
	font-size:inherit;
}

.hljs-main {
	color: #ff5252
}

.hljs-built_in {
  color: #63ff5b;
}

.hljs-comment, .hljs-quote, .hljs-deletion, .hljs-meta {
	color: #454545;
}

.hljs-params{
	color: #03A9F4;
}

.hljs-meta {
	color: #AE81FF;
}

.hljs-string {
	color: #FFFF00;
}

.strong {
	color: #F92672!important;
	font-weight: 800;
}

ul {
	margin-left: 0;
	padding-left:1em;
}

ol {
	margin-left: 0;
	padding-left:1em;
}

.reveal .code-wrapper code {
 st white-space: unset;
}

.reveal pre {
white-space: pre-wrap;
}

.reveal sup {
	font-size:0.6em;
}

</style>