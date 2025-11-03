# Llama2

This guide explains how to create and deploy a llama2 inference server and expose an API to it.
To run this example, follow these steps:

1. Install the [`kraft` CLI tool](https://unikraft.com/docs/cli/install) and a container runtime engine, for example [Docker](https://docs.docker.com/engine/install/).

2. Clone the [`examples` repository](https://github.com/unikraft-cloud/examples) and `cd` into the `examples/llama2/` directory:

```bash
git clone https://github.com/unikraft-cloud/examples
cd examples/llama2/
```

Make sure to log into Unikraft Cloud by setting your token and a [metro](https://unikraft.com/docs/platform/metros) close to you.
This guide uses `fra` (Frankfurt, 🇩🇪):

```bash
export UKC_TOKEN=token
# Set metro to Frankfurt, DE
export UKC_METRO=fra
```

When done, invoke the following command to deploy this app on Unikraft Cloud:

```bash
kraft cloud deploy -p 443:8080 -M 1024 .
```

Note that in this example the system assigns 1GB of memory.
The amount required will vary depending on the model (the section below covers how to deploy different models).

The output shows the instance address and other details:

```text
[●] Deployed successfully!
|
├────────── name: llama2-cl5bw
├────────── uuid: eddb16d4-44e7-48d6-a226-328a18745d13
├───────── state: running
├─────────── url: https://funky-rain-xds8dxbg.fra.unikraft.app
├───────── image: llama2@sha256:5af77e7381931c9f5b8f605789a238a64784b631d4b3308c5948b681c862f25a
├───── boot time: 38.29 ms
├──────── memory: 1024 MiB
├─────── service: funky-rain-xds8dxbg
├── private fqdn: llama2-cl5bw.internal
├──── private ip: 172.16.6.3
└────────── args: 8080
```

In this case, the instance name is `llama2-cl5bw` and the address is `https://funky-rain-xds8dxbg.fra.unikraft.app`.
They're different for each run.

You can retrieve a story through the `llama2` API endpoint:

```bash
curl -o - https://funky-rain-xds8dxbg.fra.unikraft.app/api/llama2
```
```text
Once upon a time, there was a little girl named Lily. She loved to eat grapes. One day, she saw a big grape on the table. Lily wanted to eat it, but she was too small. She thought, "I will try to get it when no one is looking."
The next day, Lily saw a big rock near the tower. She thought, "Maybe I can move the rock." She tried to push the rock, but it was too heavy. Lily did not give up. She tried again and again. Finally, she had a big idea. She would use a long stick to push the rock.
Lily went to the tower and pushed the rock with the stick. The rock moved! She was so happy. She picked up the grape and said, "Thank you, Rock!" Lily learned that if you are persistent and try hard, you can do anything.
```

You can list information about the instance by running:

```bash
kraft cloud instance list
```
```ansi
NAME          FQDN                                  STATE    STATUS        IMAGE                         MEMORY   VCPUS  ARGS  BOOT TIME
llama2-cl5bw  funky-rain-xds8dxbg.fra.unikraft.app  running  1 minute ago  llama2@sha256:5af77e73819...  1.0 GiB  1      8080  38286us
```

When done, you can remove the instance:

```bash
kraft cloud instance remove llama2-cl5bw
```

## Customize your app

To customize the app, update the files in the repository, listed below:

* `Kraftfile`: the Unikraft Cloud specification
* `Dockerfile`: the Docker-specified app filesystem
* `tokenizer.bin`: Exposes an API for the model
* `stories15M.bin`: The LLM model.

Lines in the `Kraftfile` have the following roles:

* `spec: v0.6`: The current `Kraftfile` specification version is `0.6`.

* `runtime: llama2`: The Unikraft runtime kernel to use is llama2.

* `rootfs: ./Dockerfile`: Build the app root filesystem using the `Dockerfile`.

* `cmd: ["8080"]`: Expose the service via port 8080

Lines in the `Dockerfile` have the following roles:

* `FROM alpine:3.14 as base`: Build the filesystem from the `alpine:3.14`, to [create a base image](https://docs.docker.com/build/building/base-images/).

* `COPY`: Copy the model and tokenizer to the Docker filesystem (to `/models`).

The following options are available for customizing the app:

* You can replace the model with others, for example from [Hugging Face](https://huggingface.co/karpathy/tinyllamas/tree/main)

* The tokenizer comes from [here](https://huggingface.co/karpathy/tinyllamas/tree/main), but feel free to replace it.

You can customize parameters for your story through a POST request on the same API endpoint.
The system recognizes the following parameters:

* `prompt`: seed the LLM with a specific string
* `model`: use specific model instead of DEFAULT
* `temperature`: valid range 0.0 - 1.0; 0.0 is deterministic, 1.0 is original (default 1.0)
* `topp`: valid range 0.0 - 1.0; top-p in nucleus sampling; 1.0 = off, 0.9 works well, but slower (default 0.9)

For example:
```bash
curl -o - https://funky-rain-xds8dxbg.fra.unikraft.app/api/llama2 -d '{ "model": "stories15M", "temperature": 0.95, "topp": 0.8, "prompt": "There once was a monkey named Bobo." }'
```

## Learn more

Use the `--help` option for detailed information on using Unikraft Cloud:

```bash
kraft cloud --help
```

Or visit the [CLI Reference](https://unikraft.com/docs/cli/overview).
