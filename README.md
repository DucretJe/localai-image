# LocalAI Image

This repository builds multi platform images for [localai](https://localai.io/).
The desired end state is to be able to run `localai` on an Apple Silicon Mx computer using docker.

## How to

### K8sGPT

* Run the image

```sh
docker run --rm --name localai -d -v ~/models:/models localai:v1.40.0 --threads 16 --models-path /model
```

* Load a model
```sh
curl http://localai.orb.local/models/apply -H "Content-Type: application/json" -d '{
    "id": "huggingface@thebloke__open-llama-13b-open-instruct-ggml__open-llama-13b-open-instruct.ggmlv3.q3_k_m.bin"
   }'
```

* Setup k8sgpt parameters
```yaml
ai:
    providers:
        - name: localai
          model: thebloke__open-llama-13b-open-instruct-ggml__open-llama-13b-open-instruct.ggmlv3.q3_k_m.bin
          baseurl: http://localai.orb.local:8080/v1
          temperature: 0.7
    defaultprovider: "localai"
kubeconfig: ""
kubecontext: ""
```

* Run k8sgpt
```sh
docker run --rm --name k8sgpt --network host -v ~/Desktop/k8sgpt.yaml:/home/root/.k8sgpt.yaml -v ~/.kube/config:/home/root/config ghcr.io/k8sgpt-ai/k8sgpt:dev-202311291513 analyze --backend localai --kubeconfig "/home/root/config" --config "/home/root/.k8sgpt.yaml" -e
```

> Current time on M3 Max v1.40.0 : 55s first run (1 issue), 45s second run

### Chat

* Run the image

```sh
docker run --rm --name localai -d -v ~/models:/models localai:v1.40.0 --threads 16 --models-path /model
```

* Load a model
```sh
curl http://localai.orb.local/models/apply -H "Content-Type: application/json" -d '{
    "id": "huggingface@thebloke__open-llama-13b-open-instruct-ggml__open-llama-13b-open-instruct.ggmlv3.q3_k_m.bin"
   }'

* Run a query

```sh
curl http://localai.orb.local/v1/completions -H "Content-Type: application/json" -d '{
     "model": "thebloke__open-llama-13b-open-instruct-ggml__open-llama-13b-open-instruct.ggmlv3.q3_k_m.bin",
     "prompt": "A long time ago in a galaxy far, far away",
     "temperature": 0.7
   }'
```
> Current time on M3 Max v1.40.0 : 26s first run, 25s second run

## Working models

* [Falcon-40b](https://huggingface.co/YokaiKoibito/falcon-40b-GGUF/tree/main): `falcon-40b-Q4_K_S.gguf` Slow but working for chat (broken for k8sgpt)

| Model    | ChatBot    | K8sGPT    |
|---------------- | --------------- | --------------- |
| [Falcon-40b](https://huggingface.co/YokaiKoibito/falcon-40b-GGUF)   `falcon-40b-Q4_K_S.gguf` | OK but slow    | NOK    |
| [Llama-13b](TheBloke/open-llama-13b-open-instruct-GGML) `open-llama-13b-open-instruct.ggmlv3.q3_K_S.bin
` |  | OK |

