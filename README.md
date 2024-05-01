# Spleen data downloader

[![Version](https://img.shields.io/docker/v/fnndsc/pl-spleendatads?sort=semver)](https://hub.docker.com/r/fnndsc/pl-spleendatadsds)
[![MIT License](https://img.shields.io/github/license/fnndsc/pl-spleendatads)](https://github.com/FNNDSC/pl-spleendatads/blob/main/LICENSE)
[![ci](https://github.com/FNNDSC/pl-spleendatads/actions/workflows/ci.yml/badge.svg)](https://github.com/FNNDSC/pl-spleendatads/actions/workflows/ci.yml)

`pl-spleendatads` is a [_ChRIS_](https://chrisproject.org/) _DS_ plugin which downloads an exemplar spleen dataset useful for training and inference experiments.

## Abstract

This is a simple _DS_ plugin suitable for training and inference on 3D spleen NiFTI volumes, as part of the [MONAI spleen segmentation exemplar notebook](https://github.com/Project-MONAI/tutorials/blob/main/3d_segmentation/spleen_segmentation_3d.ipynb). _DS_ plugins are suitable as non-root nodes of ChRIS compute trees, i.e. nodes that have a parent node. If you need a _root node_ spleen data origin, use the [companion _FS_ spleen data node](https://github.com/FNNDSC/pl-spleendata).

By default, the download is pretty big -- 1.2Gb, so make sure you have time and space. It is possible to post-download prune this. For example, if you are only interested in _training_, you can use a `--trainingOnly` flag which will prune out the 43Mb of testing NiFTI volumes. Conversely, if you are just interested in _inference_, the `--testingOnly` will remove the post download 1.2Gb of training data, saving lots of space.

You still need to download the whole set, however, before you can prune.

## Installation

`pl-spleendatads` is a _[ChRIS](https://chrisproject.org/) plugin_, meaning it can run from either within _ChRIS_ or the command-line.

## Local Usage

### On the metal

If you have checked out the repo, you can simply run `spleendatads` using

```shell
source venv/bin/activate
pip install -U ./
spleendatads input/ output/
```

### PyPI

Alternatively, you can just do a

```shell
pip install spleendatads
```

to get directly from PyPI.

### apptainer

The recommended way is to use [Apptainer](https://apptainer.org/) (a.k.a. Singularity) to run `pl-spleendatads` as a container:

```shell
apptainer exec docker://fnndsc/pl-spleendatads spleendatads [--args values...] input/ output/
```

To print its available options, run:

```shell
apptainer exec docker://fnndsc/pl-spleendatads spleendatads --help
```

## Examples

`spleendatads`, being a ChRIS _DS_ plugin, requires two positional arguments: an input directory from the upstream parent, and a directory that will contain the output data. Simply create an empty `input` and `output`.

```shell
mkdir output
apptainer exec docker://fnndsc/pl-spleendatads:latest spleendatads [--args] input/ output/
```

## Development

Instructions for developers.

### Building

Build a local container image:

```shell
docker build -t localhost/fnndsc/pl-spleendatads .
```

### Running

Mount the source code `spleendatads.py` into a container to try out changes without rebuild.

```shell
docker run --rm -it --userns=host -u $(id -u):$(id -g) \
    -v $PWD/spleendatads.py:/usr/local/lib/python3.12/site-packages/spleendatads.py:ro \
    -v $PWD/in:/incoming:ro -v $PWD/out:/outgoing:rw -w /outgoing \
    localhost/fnndsc/pl-spleendatads spleendatads /incoming /outgoing
```

### Testing

Run unit tests using `pytest`. It's recommended to rebuild the image to ensure that sources are up-to-date. Use the option `--build-arg extras_require=dev` to install extra dependencies for testing.

```shell
docker build -t localhost/fnndsc/pl-spleendatads:dev --build-arg extras_require=dev .
docker run --rm -it localhost/fnndsc/pl-spleendatads:dev pytest
```

## Release

Steps for release can be automated by [Github Actions](.github/workflows/ci.yml).
This section is about how to do those steps manually.

### Increase Version Number

Increase the version number in `setup.py` and commit this file.

### Push Container Image

Build and push an image tagged by the version. For example, for version `1.2.3`:

```
docker build -t docker.io/fnndsc/pl-spleendatads:1.2.3 .
docker push docker.io/fnndsc/pl-spleendatads:1.2.3
```

### Get JSON Representation

Run [`chris_plugin_info`](https://github.com/FNNDSC/chris_plugin#usage)
to produce a JSON description of this plugin, which can be uploaded to _ChRIS_.

```shell
docker run --rm docker.io/fnndsc/pl-spleendatads:1.2.3 chris_plugin_info -d docker.io/fnndsc/pl-spleendatads:1.2.3 > chris_plugin_info.json
```

Intructions on how to upload the plugin to _ChRIS_ can be found here:
https://chrisproject.org/docs/tutorials/upload_plugin

_-30-_
