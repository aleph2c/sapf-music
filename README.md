# SAPF-MUSIC

Sound as Pure Form.

## Training Resources

- [sapf training on youtube](https://www.youtube.com/watch?v=FY2WYXOdXoM)
- [sapf](https://github.com/lfnoise/sapf)

## Building sapf for Linux

```bash
cd ~
sudo apt update
sudo apt-get install build-essential libsndfile1-dev libhidapi-dev libudev-dev
git clone https://github.com/ahihi/sapf.git
cd sapf
git submodule init
git submodule update
sudo ./.github/scripts/install-debian-deps.sh
meson setup --buildtype release build
meson compile -C build
```

To confirm ``sapf`` built:

```bash
./build/sapf
```

Now move the built program into ``/usr/bin``

```
cp ./build/sapf /usr/bin/sapf
```


## Setting up the Environment

```bash
cd ~/projects
git clone git@github.com:aleph2c/sapf-music.git
cd sapf-musc
python3 -m venv venv
source ./venv/activate
pip install --upgrade pip
pip install -r requirements.txt
```

Now add the following to your ``~/.bashrc``

```bash
source ~/.sapf.rc
```

Where your ``.sapf.rc`` file looks like this:

```bash
export SAPF_EXPORT_ROOT=~/projects/sapf-music
export SAPF_HISTORY=~${SAPF_EXPORT_ROOT}/sapf-history.txt
export SAPF_LOG=${SAPF_EXPORT_ROOT}/sapf-log.txt
export SAPF_PRELUDE=${SAPF_EXPORT_ROOT}/sapf-prelude.txt
export SAPF_EXAMPLES=${SAPF_EXPORT_ROOT}/sapf-examples.txt
export SAPF_README="$HOME/sapf/README.txt"
export SAPF_RECORDINGS=${SAPF_EXPORT_ROOT}/recordings
export SAPF_SPECTROGRAMS=${SAPF_EXPORT_ROOT}/spectrograms
```


