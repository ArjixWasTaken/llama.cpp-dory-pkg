# llama.cpp-dory-pkg

Arch Linux package for llama.cpp with **NVIDIA Nemotron-Labs Dory** support
([ggml-org/llama.cpp#30306](https://github.com/ggml-org/llama.cpp/pull/30306),
pinned to `98a74c97`, the revision the model card uses), built for
**ROCm gfx1031** (RX 6700 XT) + Vulkan.

It is a drop-in replacement for `llama.cpp-gfx1031` (same provides/conflicts,
`replaces=` it) and ships a socket-activated server.

GitHub Actions builds the package in an `archlinux:base-devel` container and
attaches it to a release. Runners have no GPU, so CI only compiles.

## Install

```sh
gh release download --repo ArjixWasTaken/llama.cpp-dory-pkg --pattern '*.pkg.tar.zst'
sudo pacman -U llama.cpp-dory-gfx1031-*.pkg.tar.zst
sudo gpasswd -a "$USER" llm        # then log out/in
sudo systemctl enable --now llama.cpp.socket
```

## How it runs

```
client -> 127.0.0.1:8080  llama.cpp.socket
            -> llama.cpp-proxy.service   systemd-socket-proxyd, exits after 15 min idle
                 -> llama.cpp.service    llama-server router on 127.0.0.1:18080
                                         (StopWhenUnneeded: stops with the proxy)
```

- Models: every `*.gguf` in `/opt/models` (group `llm`, setgid, default ACL,
  so members can add files without sudo). Multi-file models go in a subdirectory.
- Per-model settings: `/etc/llama.cpp/models.ini` (section name = model id).
- Extra server flags: `/etc/conf.d/llama.cpp`. These override the preset file.
- Loaded models are unloaded after `--sleep-idle-seconds`; the whole server stops
  after 15 minutes without connections and starts again on the next request.

## Add a model

```sh
curl -L -o /opt/models/NVIDIA-Nemotron-Labs-Dory1.0-4B-Q5_K_M.gguf \
  https://huggingface.co/nvidia/NVIDIA-Nemotron-Labs-Dory1.0-4B-GGUF/resolve/main/NVIDIA-Nemotron-Labs-Dory1.0-4B-Q5_K_M.gguf
echo "96080be49ffd67b916018b7bdef8c484762576fb52780e10ab98ac4978a660e0  /opt/models/NVIDIA-Nemotron-Labs-Dory1.0-4B-Q5_K_M.gguf" | sha256sum -c -
```

The model list is read when the server starts. It restarts by itself after
idling; to pick up a new file right away: `sudo systemctl restart llama.cpp`.

## Dory loops and KV cache

`models.ini` sets `dory.num_loops=4` with `recurrent_kv_cache_mode=last_loop`
(~36 KiB KV per token, about 5-6 GB VRAM at 32K context). `per_loop` matches
the reference numbers more closely but needs ~96 KiB per token at 4 loops.
Valid loop counts are 1-4.

## Going back to upstream

Once #30306 is merged and released, install a regular llama.cpp package; it
conflicts with this one and replaces it. Re-run the workflow after an Arch
ROCm update that bumps library sonames.
