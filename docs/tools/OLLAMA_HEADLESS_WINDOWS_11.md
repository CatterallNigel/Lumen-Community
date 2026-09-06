# Running Ollama Headlessly on Windows 11

Ollama can be started from a Windows CMD prompt without opening its desktop UI,
leaving a console window occupied or displaying server output.

## Start Ollama

First, completely quit any Ollama desktop/tray instance. Then run:

```cmdpowershell -NoProfile -Command "$p = Start-Process -FilePath 'ollama.exe' -ArgumentList 'serve' -WindowStyle Hidden -PassThru; $p.Id | Set-Content \"$env:LOCALAPPDATA\Ollama\ollama.pid\""
```

This starts `ollama serve` as a hidden background process and immediately returns
control to the CMD prompt.

## Verify Availability

```cmd
curl.exe http://127.0.0.1:11434/api/tags
```

## Start with Logs and PID Tracking

```cmd
powershell -NoProfile -Command "$p = Start-Process -FilePath 'ollama.exe' -ArgumentList 'serve' -WindowStyle Hidden -RedirectStandardOutput \"$env:LOCALAPPDATA\Ollama\headless-out.log\" -RedirectStandardError \"$env:LOCALAPPDATA\Ollama\headless-error.log\" -PassThru; $p.Id | Set-Content \"$env:LOCALAPPDATA\Ollama\headless.pid\""
```

This records separate output and error logs and saves the server process ID.

## Stop the Recorded Instance

```cmd
powershell -NoProfile -Command "$pidToStop = Get-Content \"$env:LOCALAPPDATA\Ollama\ollama.pid\"; Stop-Process -Id $pidToStop"
```

## Lumen Boundary

Ollama remains an independently operated external service. Lumen/Praebere checks
whether the configured Ollama endpoint is available but does not start or stop the
Ollama process.

# **Pi Configuration** 

C:\Users\<username>\.pi\agent\models.json

Change the "ollama" default port from 11434 to 11435 

{
  "providers": {
    "ollama": {
      "api": "openai-completions",
      "apiKey": "ollama",
      "baseUrl": "http://127.0.0.1:11435/v1",
      "models": [
        {
          "_launch": true,
          "contextWindow": 32768,
          "id": "qwen2.5-coder:14b",
          "input": ["text"]
        },
        {
          "_launch": true,
          "contextWindow": 32768,
          "id": "qwen2.5-coder:14b-8k",
          "input": ["text"]
        },
        {
          "_launch": true,
          "contextWindow": 32768,
          "id": "qwen2.5-coder:14b-16k",
          "input": ["text"]
        },
        {
          "_launch": true,
          "contextWindow": 32768,
          "id": "qwen2.5-coder:14b-32k",
          "input": ["text"]
        }
      ]
    }
  }
}