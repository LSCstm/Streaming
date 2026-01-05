# Streaming

<div align="center">
  <a href="https://github.com/osthread/">
    <img src="og_trinix.png" alt="Trinix logo" width="150" height="80" />
  </a>

  <h3 align="center">Watch trending movies and TV shows for free</h3>

  <p align="center">
    A lightweight Flask application that surfaces curated streaming links powered by The Movie Database (TMDB).
    <br />
  </p>
</div>

## Overview

Trinix Free-Streaming pairs a minimal Flask backend with HTML templates to deliver an always up-to-date catalog of movies and television shows. The service queries TMDB for trending, upcoming, and searched titles, verifies that each item can be streamed, and renders the results in a clean, responsive interface.

The project was designed to complement the Trinix Discord bot but also works as a standalone web application you can host for friends or personal use.

> **Disclaimer**
> Trinix Free-Streaming simply aggregates publicly available links. You are responsible for complying with local laws and the terms of service of the content providers you access through this project.

## Features

- **Trending dashboards** – Display daily trending movies and TV series from TMDB.
- **Upcoming releases** – Highlight films scheduled to hit theaters soon.
- **Search experience** – Look up any movie or show using TMDB's multi-search endpoint.
- **Link validation** – Automatically confirm that each title has a working embed on the configured streaming provider before it is shown to viewers.
- **Watch page** – Stream the selected title with contextual metadata (synopsis, poster, release information).
- **Systemd-friendly deployment** – Includes instructions for running the application as a managed Linux service for continuous availability.

## Project structure

```
Free-Streaming/
├── main.py                # Flask entry point served through Waitress
├── Modules/
│   └── Api/
│       └── searchapi.py   # TMDB + streaming provider integration
├── templates/             # Jinja2 templates for the UI
├── static/                # Stylesheets, scripts, and assets for the front-end
└── og_trinix.png          # Project logo
```

## Requirements

- Python 3.9+
- pip
- TMDB API key (required for all API calls)

Optional tooling for production deployments:

- systemd (PID 1 service manager on most modern Linux distributions)

## Getting started

1. **Clone the repository**
   ```bash
   git clone https://github.com/osthread/Free-Streaming.git
   cd Free-Streaming
   ```

2. **Create a virtual environment (recommended)**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Install Python dependencies**
   ```bash
   pip install flask flask-cors waitress requests
   ```

4. **Configure your TMDB API key**
   - Open `Modules/Api/searchapi.py` and replace the placeholder string `API-Key` with `Bearer <your_tmdb_v4_api_key>`.
   - You can alternatively load the key from an environment variable and adjust the code to read it securely.

5. **Start the development server**
   ```bash
   python main.py
   ```
   The application runs with Waitress on `http://127.0.0.1:9999/watch/` by default.

## Deployment with systemd (optional)

On Linux servers that use systemd, you can run the application as a managed service:

1. **Create a dedicated user (optional but recommended)**
   ```bash
   sudo useradd --system --home /opt/trinix --shell /usr/sbin/nologin trinix
   sudo mkdir -p /opt/trinix
   sudo chown trinix:trinix /opt/trinix
   ```

2. **Copy the project files**
   ```bash
   sudo rsync -av --exclude '.venv' ./ /opt/trinix/
   ```

3. **Create a virtual environment and install dependencies**
   ```bash
   sudo -u trinix python3 -m venv /opt/trinix/.venv
   sudo -u trinix /opt/trinix/.venv/bin/pip install flask flask-cors waitress requests
   ```

4. **Create the systemd service unit**
   Create `/etc/systemd/system/trinix.service` with the following contents:
   ```ini
   [Unit]
   Description=Trinix Free-Streaming service
   After=network-online.target
   Wants=network-online.target

   [Service]
   Type=simple
   User=trinix
   WorkingDirectory=/opt/trinix
   Environment="PATH=/opt/trinix/.venv/bin"
   ExecStart=/opt/trinix/.venv/bin/python main.py
   Restart=on-failure

   [Install]
   WantedBy=multi-user.target
   ```

5. **Enable and start the service**
   ```bash
   sudo systemctl daemon-reload
   sudo systemctl enable --now trinix.service
   sudo systemctl status trinix.service
   ```

With this configuration systemd will restart the application automatically if it exits unexpectedly. Use `journalctl -u trinix.service` to review logs.

## Customization tips

- **Change the streaming provider** – Update the `check_movie` and `check_media` methods in `Modules/Api/searchapi.py` to point to a different embed service.
- **Adjust filtering rules** – Modify `required_keys` in `modified_results` if you want to display additional metadata.
- **Style the UI** – Edit files inside the `static/` directory or the Jinja templates in `templates/`.

## Contributing

Pull requests are welcome. Please open an issue to discuss major changes before submitting a PR, and make sure to test your updates locally.

## License

Distributed under the GPL-3.0 License. See [`LICENSE`](LICENSE) for full details.

## Acknowledgements

- Logo by [gh0st_artz](https://www.instagram.com/gh0st_artz/)
- Movie and TV metadata provided by [The Movie Database (TMDB)](https://www.themoviedb.org/)
