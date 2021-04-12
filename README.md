# Text Renderer
Generate text images for training deep learning OCR model (e.g. [CRNN](https://github.com/bgshih/crnn)). ![example](./image/example.gif)

- [x] Modular design. You can easily add [Corpus](https://oh-my-ocr.github.io/text_renderer/corpus/index.html), [Effect](https://oh-my-ocr.github.io/text_renderer/effect/index.html), [Layout](https://oh-my-ocr.github.io/text_renderer/layout/index.html).
- [x] Support generate `lmdb` dataset which compatible with [PaddleOCR](https://github.com/PaddlePaddle/PaddleOCR), see [Dataset](https://oh-my-ocr.github.io/text_renderer/dataset.html)
- [x] Support render multi corpus on image with different font, font size or font color. [Layout](https://oh-my-ocr.github.io/text_renderer/layout/index.html) is responsible for the layout between multiple corpora
- [ ] Generate vertical text
- [ ] Corpus sampler: helpful to perform character balance

[Documentation](https://oh-my-ocr.github.io/text_renderer/index.html)

## Run Example

Run following command to generate images using example data:

```bash
git clone https://github.com/oh-my-ocr/text_renderer
cd text_renderer
python3 setup.py develop
pip3 install -r docker/requirements.txt
python3 main.py \
    --config example_data/example.py \
    --dataset img \
    --num_processes 2 \
    --log_period 10
```

The data is generated in the `example_data/output` directory.

You can check config file [example_data/example.py](https://github.com/oh-my-ocr/text_renderer/blob/master/example_data/example.py) to learn how to use text_renderer,
or follow the [Quick Start](https://github.com/oh-my-ocr/text_renderer#quick-start) to learn how to setup configuration
 

## Quick Start
### Prepare file resources
   
- Font files: `.ttf`、`.otf`、`.ttc`
- Background images of any size, either from your business scenario or from publicly available datasets ([COCO](https://cocodataset.org/#home), [VOC](http://host.robots.ox.ac.uk/pascal/VOC/))
- Corpus: text_renderer offers a wide variety of [text sampling methods](https://oh-my-ocr.github.io/text_renderer/corpus/index.html), 
to use these methods, you need to consider the preparation of the corpus from two perspectives：
1. The corpus must be in the target language for which you want to perform OCR recognition
2. The corpus should meets your actual business needs, such as education field, medical field, etc.
- Charset file [Optional but recommend]: OCR models in real-world scenarios (e.g. CRNN) usually support only a limited character set, 
so it's better to filter out characters outside the character set during data generation. 
You can do this by setting the [chars_file](https://oh-my-ocr.github.io/text_renderer/corpus/char_corpus.html) parameter

You can download pre-prepared file resources for this `Quick Start` from here: 

- [simsun.ttf](https://github.com/oh-my-ocr/text_renderer/raw/master/example_data/font/simsun.ttf)
- [background.png](https://github.com/oh-my-ocr/text_renderer/raw/master/example_data/bg/background.png)
- [eng_text.txt](https://github.com/oh-my-ocr/text_renderer/raw/master/example_data/text/eng_text.txt)

Save these resource files in the same directory:
```
workspace
├── bg
│ └── background.png
├── corpus
│ └── eng_text.txt
└── font
    └── simsun.ttf
```

### Create config file
Create a `config.py` file in `workspace` directory. One configuration file must have a `configs` variable, it's 
a list of [GeneratorCfg](https://oh-my-ocr.github.io/text_renderer/config.html#text_renderer.config.GeneratorCfg). 

The complete configuration file is as follows:
```python
import os
from pathlib import Path

from text_renderer.effect import *
from text_renderer.corpus import *
from text_renderer.config import (
    RenderCfg,
    NormPerspectiveTransformCfg,
    GeneratorCfg,
    SimpleTextColorCfg,
)

CURRENT_DIR = Path(os.path.abspath(os.path.dirname(__file__)))


def story_data():
    return GeneratorCfg(
        num_image=10,
        save_dir=CURRENT_DIR / "output",
        render_cfg=RenderCfg(
            bg_dir=CURRENT_DIR / "bg",
            height=32,
            perspective_transform=NormPerspectiveTransformCfg(20, 20, 1.5),
            corpus=WordCorpus(
                WordCorpusCfg(
                    text_paths=[CURRENT_DIR / "corpus" / "eng_text.txt"],
                    font_dir=CURRENT_DIR / "font",
                    font_size=(20, 30),
                    num_word=(2, 3),
                ),
            ),
            corpus_effects=Effects(Line(0.9, thickness=(2, 5))),
            gray=False,
            text_color_cfg=SimpleTextColorCfg(),
        ),
    )


configs = [story_data()]
```

In the above configuration we have done the following things:

1. Specify the location of the resource file
2. Specified text sampling method: 2 or 3 words are randomly selected from the corpus
3. Configured some effects for generation
   - Perspective transformation [NormPerspectiveTransformCfg](https://oh-my-ocr.github.io/text_renderer/_modules/text_renderer/config.html#NormPerspectiveTransformCfg)
   - Random [Line Effect](https://oh-my-ocr.github.io/text_renderer/effect/line.html)
   - Fix output image height to 32
   - Generate color image. `gray=False`, `SimpleTextColorCfg()`
4. Specifies font-related parameters: `font_size`, `font_dir`

### Run 
Run `main.py`, it only has 4 arguments:
- config：Python config file path
- dataset: Dataset format `img` or `lmdb`
- num_processes: Number of processes used
- log_period: Period of log printing. (0, 100)

See more Corpus, Effect, Layout you can use in the [documentation](https://oh-my-ocr.github.io/text_renderer/index.html)

## Contribution

- Corpus: Feel free to contribute more corpus generators to the project, 
  It does not necessarily need to be a generic corpus generator, but can also be a business-specific generator, 
  such as generating ID numbers


## Run in Docker

Build image

```bash
docker build -f docker/Dockerfile -t text_renderer .
```

Config file is provided by `CONFIG` environment.
In `example.py` file, data is generated in `example_data/output` directory,
so we map this directory to the host.

```bash
docker run --rm \
-v `pwd`/example_data/docker_output/:/app/example_data/output \
--env CONFIG=/app/example_data/example.py \
--env DATASET=img \
--env NUM_PROCESSES=2 \
--env LOG_PERIOD=10 \
text_renderer
```

## Build docs

```bash
cd docs
make html
```

Open `_build/html/index.html`


## Citing text_renderer
If you use text_renderer in your research, please consider use the following BibTeX entry.

```BibTeX
@misc{text_renderer,
  author =       {oh-my-ocr},
  title =        {text_renderer},
  howpublished = {\url{https://github.com/oh-my-ocr/text_renderer}},
  year =         {2021}
}
```
