---
title: "Custom servers"
description: "Connect kunji to LM Studio, vLLM, llama.cpp server, LiteLLM or any OpenAI-compatible API server."
sidebar:
  order: 10
---

Any server that implements the OpenAI chat-completions API can work with kunji. This guide
shows how to start three popular local servers and register them with kunji.

The examples use local-only addresses. Keep the server bound to your own machine unless you
have configured authentication and a secure network path. Replace angle-bracket placeholders
such as `<model>` before running a command; they are not literal values.

## LM Studio

1. Open LM Studio and load the model you want to use.
2. Start its local API server:

   ```bash
   lms server start --port 1234
   ```

3. Check that the server is running:

   ```bash
   lms server status
   ```

4. Add it to kunji:

   ```bash
   kunji providers add lmstudio --base-url http://127.0.0.1:1234/v1 --no-key --name "LM Studio"
   kunji chat -m lmstudio:<model-id>
   ```

Replace `<model-id>` with the model identifier exposed by your LM Studio server.

See the [LM Studio server CLI reference](https://lmstudio.ai/docs/cli/serve/server-start).

## vLLM

Start the OpenAI-compatible server with the model you want to serve:

```bash
vllm serve <model> --host 127.0.0.1 --port 8000
```

Replace `<model>` with a model name or local model path supported by your vLLM installation.
Then register the server with kunji:

```bash
kunji providers add vllm --base-url http://127.0.0.1:8000/v1 --no-key --name "vLLM"
kunji chat -m vllm:<model-id>
```

When vLLM is started with `--api-key`, omit `--no-key` and add the key locally:

```bash
kunji providers add vllm --base-url http://127.0.0.1:8000/v1 --name "vLLM"
kunji keys add vllm
```

Use the model identifier returned by the server in `kunji chat`. See the
[vLLM CLI guide](https://docs.vllm.ai/en/latest/cli/).

**Security note:** vLLM's `--api-key` protects the documented OpenAI-compatible API path
prefixes, not every endpoint on the server. Keep this example bound to loopback. For remote
access, put vLLM behind a reverse proxy that restricts exposed endpoints and follow the
[vLLM security guidance](https://docs.vllm.ai/en/latest/usage/security/).

## llama.cpp server

Start `llama-server` with a GGUF model file. This example explicitly selects port 8080:

```bash
llama-server -m /path/to/model.gguf --host 127.0.0.1 --port 8080
```

In another terminal, register it with kunji:

```bash
kunji providers add llamacpp --base-url http://127.0.0.1:8080/v1 --no-key --name "llama.cpp"
kunji chat -m llamacpp:<model-id>
```

Replace `/path/to/model.gguf` with the path to your model file and use the model identifier
reported by the server. See the
[llama.cpp server README](https://github.com/ggml-org/llama.cpp/tree/master/tools/server).

## Troubleshooting

- **Connection refused:** make sure the server is running, then check its log for the address
  and port it is listening on. The port in `--base-url` must match that address.
- **Wrong API URL:** the base URL must end in `/v1`. Do not add
  `/chat/completions`; kunji builds the chat-completions endpoint itself.
- **Authentication error:** remove `--no-key` when adding a server that requires a key, then
  run `kunji keys add <id>` to store it in the keychain.
- **Model not found:** use a model identifier reported by that server, not necessarily the
  model's display name in its desktop UI.
- **Port already in use:** choose another port when starting the server and update the
  `--base-url` to use the same port.

For any server that is not confined to your own computer or trusted local network, use HTTPS
and configure authentication. Do not expose an unauthenticated inference server directly to
the public internet.

## Add another compatible server

Use the same pattern for LiteLLM or another OpenAI-compatible server:

```bash
kunji providers add <id> --base-url http://127.0.0.1:<port>/v1 --no-key --name "My server"
kunji chat -m <id>:<model-id>
```

- **id**: a short name using lowercase letters, numbers and dashes.
- **`--base-url`**: the server address ending in `/v1`.
- **`--no-key`**: use only when the server does not require authentication. Otherwise omit
  this flag and run `kunji keys add <id>`.

From the menu: **Providers and keys → Add a custom server**.

## Remove a server

```bash
kunji providers remove lmstudio
```

## Point a built-in provider somewhere else

To send a built-in provider through a proxy, set its `base_url`:

```toml
[providers.openai]
base_url = "https://my-proxy.example.com/v1"
```
