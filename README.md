# Kodi integration for Remote Two

Using [pykodi](https://github.com/OnFreund/PyKodi)
and [uc-integration-api](https://github.com/aitatoi/integration-python-library)

The driver lets discover and configure your Kodi instances (discovery not supported yet). A media player entity is exposed to the core.

Supported attributes:
- State (on, off, playing, paused, unknown)
- Title
- Album
- Artist
- Artwork
- Media position / duration
- Volume (level and up/down) and mute


Supported commands:
- Turn off (turn on is not supported)
- Direction pad and enter
- Numeric pad
- Back
- Next
- Previous
- Volume up
- Volume down
- Pause / Play
- Channels Up/Down
- Menus (home, context)
- Colored buttons
- Subtitle/audio language switching
- Fast forward / rewind
- Simple commands (more can be added) : video menu, toggle fullscreen, zoom in/out, increase/decrease aspect ratio, toggle subtitles, subtitles delay minus/plus, audio delay minus/plus

## Usage

- Kodi must be running for setup, and control enabled from Settings > Services > Control section. Set the username, password and enable HTTP control.
- Port numbers shouldn't be modified normally (8080 for HTTP and 9090 for websocket) : websocket port is not configurable from the GUI (in advanced settings file)
- There is no turn on command : Kodi has to be started some other way

## To do

- ~~Add automatic discovery of Kodi instances on the network~~ done
- Add more simple commands if necessary


### Setup

- Requires Python 3.11
- Under a virtual environment : the driver has to be run in host mode and not bridge mode, otherwise the turn on function won't work (a magic packet has to be sent through network and it won't reach it under bridge mode)
- Your Kodi instance has to be started in order to run the setup flow and process commands. When configured, the integration will detect automatically when it will be started and process commands.
- Install required libraries:  
  (using a [virtual environment](https://docs.python.org/3/library/venv.html) is highly recommended)

```shell
pip3 install -r requirements.txt
```

For running a separate integration driver on your network for Remote Two, the configuration in file
[driver.json](driver.json) needs to be changed:

- Set `driver_id` to a unique value, `uc_kodi_driver` is already used for the embedded driver in the firmware.
- Change `name` to easily identify the driver for discovery & setup with Remote Two or the web-configurator.
- Optionally add a `"port": 8090` field for the WebSocket server listening port.
    - Default port: `9090`
    - Also overrideable with environment variable `UC_INTEGRATION_HTTP_PORT`

### Run

```shell
python3 intg-kodi/driver.py
```

See
available [environment variables](https://github.com/unfoldedcircle/integration-python-library#environment-variables)
in the Python integration library to control certain runtime features like listening interface and configuration
directory.

## Build self-contained binary for Remote Two

After some tests, turns out python stuff on embedded is a nightmare. So we're better off creating a single binary file
that has everything in it.

To do that, we need to compile it on the target architecture as `pyinstaller` does not support cross compilation.

### x86-64 Linux

On x86-64 Linux we need Qemu to emulate the aarch64 target platform:

```bash
sudo apt install qemu binfmt-support qemu-user-static
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes
```

Run pyinstaller:

```shell
docker run --rm --name builder \
    --platform=aarch64 \
    --user=$(id -u):$(id -g) \
    -v "$PWD":/workspace \
    docker.io/unfoldedcircle/r2-pyinstaller:3.11.6  \
    bash -c \
      "python -m pip install -r requirements.txt && \
      pyinstaller --clean --onefile --name intg-kodi intg-kodi/driver.py"
```

### aarch64 Linux / Mac

On an aarch64 host platform, the build image can be run directly (and much faster):

```shell
docker run --rm --name builder \
    --user=$(id -u):$(id -g) \
    -v "$PWD":/workspace \
    docker.io/unfoldedcircle/r2-pyinstaller:3.11.6  \
    bash -c \
      "python -m pip install -r requirements.txt && \
      pyinstaller --clean --onefile --name intg-kodi intg-kodi/driver.py"
```

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the
[tags and releases in this repository](https://github.com/albaintor/integration-kodi/releases).

## Changelog

The major changes found in each new release are listed in the [changelog](CHANGELOG.md)
and under the GitHub [releases](https://github.com/albaintor/integration-kodi/releases).

## Contributions

Please read our [contribution guidelines](CONTRIBUTING.md) before opening a pull request.

## License

This project is licensed under the [**Mozilla Public License 2.0**](https://choosealicense.com/licenses/mpl-2.0/).
See the [LICENSE](LICENSE) file for details.
