# J-CRe3: Japanese Conversation dataset for Real-world Reference Resolution

[![Conference](https://img.shields.io/badge/LREC--COLING-2024-bbeaff.svg)](https://arxiv.org/abs/2403.19259)
[![arXiv](https://img.shields.io/badge/arXiv-2403.19259-b31b1b.svg)](https://arxiv.org/abs/2403.19259)


[[Paper (ja)]](https://www.anlp.jp/proceedings/annual_meeting/2023/pdf_dir/H12-1.pdf)
[Paper (en)]

![Dataset Overview](https://raw.githubusercontent.com/riken-grp/J-CRe3/main/docs/overview.png)

## Overview

J-CRe3 is a real-world Japanese conversation dataset that contains egocentric video, third-person video, and dialogue
audio of real-world
conversations between two people.
The conversations involve a robot that is helping its master with daily mundane tasks, including many object
manipulations.
It contains 93 scenario-based dialogues with 2,131 utterances and 11,024 seconds of video.
As shown in the figure above, the dataset is annotated with the following information:

- **Bounding boxes**: The bounding boxes of the objects and regions in the video frames. Objects and regions that were
  referred to in the dialogue are annotated. Each bounding box has a class name and an instance ID. This dataset
  contains 79,694 bounding boxes in total.

- **Textual references**: The text-to-text references in the dialogue. The annotations include predicate-argument
  structures, bridging references, and coreferences.

- **Text-to-object references**: The text-to-object references between phrases in dialogue texts and objects in video
  frames. The annotations include indirect reference relations, such as predicate-argument structures and bridging
  references as well as direct reference relations.

## Video and Audio Files

The egocentric video, third-person video, and dialogue audio files are available on the Box cloud storage.
Download `J-CRe3.zip` from <https://riken-share.box.com/s/25vjhsccliimse0q74z35nv8j5bz31is> (Password: `J-CRe3-data`) and extract it to the `recording`
directory.
The directory structure should look like this:

```plaintext
recording/
├── 20220302-56130207-1/
│   ├── images/
│   │   ├── 001.png
│   │   ├── 002.png
│   │   └── ...
│   ├── fp_video.mp4
│   ├── cam11.mp4
│   ├── cam12.mp4
│   ├── cam13.mp4
│   ├── cam14.mp4
│   ├── audio.wav
│   ├── info.json
│   └── timestamp.json
└── ...
```

- `images/`: The directory containing the egocentric video frames extracted from the `fp_video.mp4` at 1fps.
  You can use the `ffmpeg` command to extract the frames from the video.
  Note that we scaled down the resolution to half for efficient annotation.

  ```shell
  ffmpeg \
    -i recording/20220302-56130207-1/fp_video.mp4 \
    -vf "select='not(mod(n\,30))',scale=iw/2:ih/2" \
    -fps_mode passthrough \
    'recording/20220302-56130207-1/images/%03d.png'
  ```

- `fp_video.mp4`: The egocentric video file.

- `cam11.mp4`, `cam12.mp4`, `cam13.mp4`, `cam14.mp4`: The third-person video files.

- `audio.wav`: The dialogue audio file (stereo).

- `info.json`: The metadata file containing the scenario ID, the speaker's ID, and the temporal alignment between the
  utterances and the video frames.

  ```json
  {
    "scenario_id": "scenario ID (string)",
    "utterances": [
      {
        "text": "transcribed utterance (string)",
        "sids": [
          "sentence ID (string)"
        ],
        "start": "start time in milliseconds (number)",
        "end": "end time in milliseconds (number)",
        "duration": "duration in milliseconds (number)",
        "speaker": "speaker ID (主人 or ロボット)",
        "image_ids": [
          "temporally corresponding image ID (string)"
        ]
      }
    ],
    "images": [
      {
        "id": "image ID (string)",
        "path": "relative path to the image file (string)",
        "time": "timestamp in milliseconds (number)"
      }
    ]
  }
  ```

- `timestamp.json`: The timestamp file containing the start time of the recording.

  ```json
  {
    "tp_video": {
      "start": {
        "from": "time when the record start instruction was sent (string)",
        "to": "time when the record start instruction was completed (string)"
      },
      "end": {
        "from": "time when the record end instruction was sent (string)",
        "to": "time when the record end instruction was completed (string)"
      }
    },
    "fp_video": {
      "start": {
        "from": "ditto",
        "to": "ditto"
      },
      "end": {
        "from": "ditto",
        "to": "ditto"
      }
    },
    "audio": {
      "start": {
        "from": "ditto",
        "to": "ditto"
      },
      "end": {
        "from": "ditto",
        "to": "ditto"
      }
    }
  }
  ```

## Annotation Files

- `textual_annotations/`: Processed transcriptions of the dialogue audio with textual reference annotations. The files
  in this directory are in [the KNP format](https://rhoknp.readthedocs.io/en/latest/format/index.html#knp), which can be
  easily parsed by [rhoknp](https://github.com/ku-nlp/rhoknp).

- `visual_annotations/`: Visual annotations of the bounding boxes and text-to-object references in the video. The files
  in this directory are in JSON format whose schema is described below.

  ```json
  {
    "scenarioId": "scenario ID (string)",
    "images": [
      {
        "imageId": "image ID (string)",
        "boundingBoxes": [
          {
            "imageId": "image ID (string)",
            "instanceId": "instance ID (string)",
            "rect": {
              "x1": "x-coordinate of the top-left corner (number)",
              "y1": "y-coordinate of the top-left corner (number)",
              "x2": "x-coordinate of the bottom-right corner (number)",
              "y2": "y-coordinate of the bottom-right corner (number)"
            },
            "className": "class name (string)"
          }
        ]
      }
    ],
    "utterances": [
      {
        "text": "raw utterance text (string)",
        "phrases": [
          {
            "text": "raw phrase text (string)",
            "relations": [
              {
                "type": "relation type (string)",
                "instanceId": "instance ID (string)"
              }
            ]
          }
        ]
      }
    ]
  }
  ```

- `id/`: Scenario ID files providing train/valid/test split.

- `transcriptions/`: Transcription of the dialogue audio. This directory contains files directly generated by the
  transcription tool (ELAN). Refer to the files under the `textual_annotations` directory for processed transcriptions.

- `raw_annotations/`: Bounding box and text-to-object reference annotations. The files in this directory are directly
  generated by the annotation tool, and most of the annotations are omitted for efficient annotation. Refer to the files
  under the `visual_annotations` directory for processed complete annotations.

## Statistics

See [statistics.md](./docs/statistics.md).

## License

The license for this dataset is subject to CC BY-SA 4.0.
<https://creativecommons.org/licenses/by-sa/4.0/>
Please cite the following paper when using this dataset.

## Citation

```bibtex
@inproceedings{植田2023a,
  author    = {植田 暢大 and 波部 英子 and 湯口 彰重 and 河野 誠也 and 川西 康友 and 黒橋 禎夫 and 吉野 幸一郎},
  title     = {実世界における総合的参照解析を目的としたマルチモーダル対話データセットの構築},
  booktitle = {言語処理学会 第29回年次大会},
  year      = {2023},
  address   = {沖縄},
}
```

```bibtex
@inproceedings{ueda-2024-jcre3,
  title={J-CRe3: A Japanese Conversation Dataset for Real-world Reference Resolution},
  author={Nobuhiro Ueda and Hideko Habe and Yoko Matsui and Akishige Yuguchi and Seiya Kawano and Yasutomo Kawanishi and Sadao Kurohashi and Koichiro Yoshino},
  booktitle={Proceedings of the 2024 Joint International Conference on Computational Linguistics, Language Resources and Evaluation (LREC-COLING 2024)},
  year={2024},
  pages={},
  address={Turin, Italy},
}
```
