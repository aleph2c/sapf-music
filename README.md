# SAPF-MUSIC

Sound as Pure Form.

## Training Resources

- [pulusound: first look at sapf](https://www.youtube.com/watch?v=jN5Ij3Cazh8)
- [pulusound: sequencing in sapf](https://www.youtube.com/watch?v=V2oRfXulHio)
- [lantertronics: sapf install (on mac)](https://www.youtube.com/watch?v=FY2WYXOdXoM)
- [lantertronics: language basic and FM Synthesis](https://www.youtube.com/watch?v=FY2WYXOdXoM)
- [sapf](https://github.com/lfnoise/sapf)
- [sox sgram replacement](https://www.youtube.com/watch?v=k1TRq5lk0Sg)

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
export SAPF_SPECTROGRAMS=${[SAPF_EXPORT_ROOT](SAPF_EXPORT_ROOT)}/spectrograms
```
## Configuring nvim to run a .sapf file


In your ``~/.config/nvim/lua/sapf_player.lua`` add the following:

```lua
-- ~/.config/nvim/lua/sapf_player.lua

local M = {}

-- The corrected command. 'sleep infinity' keeps the pipe open without
-- trying to read from the terminal, which prevents the shell from stopping it.
local SAPF_CMD_TEMPLATE = "((echo '%s'; sleep infinity) | sapf) &"

local function get_code()
  local mode = vim.api.nvim_get_mode().mode
  if mode == 'v' or mode == 'V' then
    local start_pos = vim.api.nvim_buf_get_mark(0, '<')
    local end_pos = vim.api.nvim_buf_get_mark(0, '>')
    local lines = vim.api.nvim_buf_get_lines(0, start_pos[1] - 1, end_pos[1], false)
    if #lines > 0 then
      lines[#lines] = string.sub(lines[#lines], 1, end_pos[2] + 1)
      lines[1] = string.sub(lines[1], start_pos[2] + 1)
    end
    return table.concat(lines, '\n')
  else
    return table.concat(vim.api.nvim_buf_get_lines(0, 0, -1, false), '\n')
  end
end

function M.play()
  -- First, stop any currently playing instance.
  M.stop()
  -- Wait a moment for the old process to fully terminate before starting a new one.
  -- This uses the external 'sleep' command, which we know works on your system.
  vim.fn.system('sleep 0.05')

  local code = get_code()
  -- Escape single quotes in the code to prevent breaking the shell command.
  local sanitized_code = string.gsub(code, "'", "'\\''")

  if sanitized_code and #sanitized_code > 0 then
    local command_to_run = string.format(SAPF_CMD_TEMPLATE, sanitized_code)
    -- print("Executing: " .. command_to_run)
    print("<leader><leader>s to stop ")
    -- Use vim.fn.system() to run the command. The '&' makes it non-blocking.
    vim.fn.system(command_to_run)
    -- print("sapf process started in background.")
  else
    -- print("No code to play.")
  end
end

function M.stop()
  -- print("Attempting to stop all sapf processes...")
  -- Use `pkill` to send the interrupt signal (SIGINT, same as Ctrl+C)
  -- to any process named 'sapf'. This is the correct way to stop it.
  print(" ")
  vim.cmd('echo ""')
  local command = "pkill -2 sapf >/dev/null 2>&1"
  vim.fn.system(command)
  -- print("Stop signal sent to sapf.")
end

return M

```

In your ``~/.config/nvim/init.vim`` add something like this:

```vim
" --- SAPF AUDIO PLAYER ---
lua require('sapf_player')
nnoremap <Leader><Leader>e <Cmd>lua require('sapf_player').play()<CR>
vnoremap <Leader><Leader>e <Cmd>lua require('sapf_player').play()<CR>
nnoremap <Leader><Leader>s <Cmd>lua require('sapf_player').stop()<CR>
"echom "Sapf player loaded: <Leader><Leader>e (play), <Leader><Leader>s (stop)"

```

With this configuration the sapf code in an ``.sapf`` file can be played within
nvim by typing ``<Leader><Leader>e`` and it can be stopped with
``<Leader><Leader>s``

