# QR Code Generator

Quick QR code generation from the command line. This is a small Go HTTP server: send it some text or a URL and it sends back a PNG QR code.

- One endpoint, two parameters
- PNG output, black on white, medium error correction (survives about 15% damage)
- One dependency: [`skip2/go-qrcode`](https://github.com/skip2/go-qrcode)

## Requirements

- Go 1.26 or newer
- `curl`, or any other HTTP client

## Quick start

Clone the repo and start the server. It listens on port `8080`.

```bash
git clone https://github.com/NimaNaghibi143/qr-code-generator.git
cd qr-code-generator
go run .
```

In a second terminal, request a QR code:

```bash
curl -X POST \
  --form-string "size=512" \
  --form-string "content=https://example.com" \
  -o qr.png \
  http://localhost:8080/generate
```

This writes `qr.png`, a 512×512 QR code that opens `https://example.com`. Stop the server with `Ctrl+C`.

## API

### `/generate`

Accepts `GET` with query parameters, or `POST` with form fields (`multipart/form-data` or `application/x-www-form-urlencoded`).

| Parameter | Required | Description |
|-----------|----------|-------------|
| `content` | yes | What the QR code holds: a URL, plain text, a phone number, Wi-Fi details and so on. |
| `size`    | yes | Width and height of the image in pixels, such as `256` or `1024`. A negative value sets the size of each QR module instead: `-10` makes each square 10 px. |

### Responses

| Status | Content type | Body |
|--------|--------------|------|
| `200` | `image/png` | The QR code image |
| `400` | `application/json` | An error message: `content` is missing, `size` is missing or not a number, or the content is too long for a QR code |

## Examples

**From the browser.** With the server running, open this address and save the image:

```
http://localhost:8080/generate?content=https://example.com&size=512
```

**Using GET with curl.** Quote the URL so the shell leaves the `&` alone:

```bash
curl -o qr.png "http://localhost:8080/generate?content=Hello%20world&size=256"
```

**A URL that contains its own `?` or `&`.** Use POST with `--form-string`, so you don't have to escape it:

```bash
curl -X POST \
  --form-string "size=512" \
  --form-string "content=https://example.com/page?ref=qr&lang=en" \
  -o qr.png \
  http://localhost:8080/generate
```

**A Wi-Fi network.** Phones offer to join when they scan this:

```bash
curl -X POST \
  --form-string "size=512" \
  --form-string "content=WIFI:T:WPA;S:MyNetwork;P:MyPassword;;" \
  -o wifi-qr.png \
  http://localhost:8080/generate
```

## Build a binary

```bash
go build -o qr-code-generator .
./qr-code-generator
```

## Tips

- Put `https://` at the start of web addresses. Without it, some phone cameras show the address as plain text instead of opening it.
- For print, use `1024` or larger. QR codes are made of hard-edged squares, so a big PNG scales down cleanly.
- Scan the code with your phone before you print or share it.
