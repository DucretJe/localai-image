# LocalAI Image

This repository builds multi platform images for [localai](https://localai.io/).
The desired end state is to be able to run `localai` on an Apple Silicon Mx computer using docker.

## How to

> In this example we consider we have a model already downloaded in `~/Desktops/models`, here it's `GPT4all.gguf`

* Run the image

```sh
docker run --rm --name localai -v ~/Desktops/models:/models -d -ti --pull always ghcr.io/ducretje/localai-image --threads 8 --models-path /models --preload-models '[{"model": "GPT4all.gguf"}]'
```

* Run a query

```sh
curl http://localai.orb.local/v1/completions -H "Content-Type: application/json" -d '{
     "model": "GPT4all.gguf",
     "prompt": "A long time ago in a galaxy far, far away",
     "temperature": 0.7
   }'
```
