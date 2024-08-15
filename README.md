# Azure setup

Create an Azure OpenAI service name thing and then go into the workspace and create a model, which has a deployment name. Now add the following to your bashrc.
```
export AZURE_OPENAI_API_KEY='<one of the keys for your azure openai API>'
export AZURE_OPENAI_API_BACKEND='https://<name of azure openai endpoint>.openai.azure.com/openai/deployments/<model deployment name>/chat/completions?api-version=2024-06-01'
```

## Install lua to be the main language of neovim

If you're still on vimscript like I was, do the following.

Create `~/.config/nvim/init.lua` and set it up in accordance with the structured setup in [this page](https://lazy.folke.io/installation). Also have to install some stuff (look at the requirements).

Then, move the old `init.vim` to `vimrc.vim` and then add
```
local vimrc = vim.fn.stdpath("config") .. "/vimrc.vim"
vim.cmd.source(vimrc)
```

Now create `~/.config/nvim/lua/plugins/llm.lua` and paste this into it
```
return {
  {
    'neilzxu/dingllm.nvim',
    dependencies = { 'nvim-lua/plenary.nvim' },
    config = function()
      local system_prompt =
        'You should replace the code that you are sent, only following the comments. Do not talk at all. Only output valid code. Do not provide any backticks that surround the code. Never ever output backticks like this ```. Any comment that is asking you for something should be removed after you satisfy them. Other comments should left alone. Do not output backticks'
      local helpful_prompt = 'You are a helpful assistant. What I have sent are my notes so far. You are very curt, yet helpful.'
      local dingllm = require 'dingllm'

      local function azure_openai_replace()
        dingllm.invoke_llm_and_stream_into_editor({
          url_name = 'AZURE_OPENAI_API_BACKEND',
          api_key_name = 'AZURE_OPENAI_API_KEY',
          system_prompt = system_prompt,
          replace = true,
        }, dingllm.make_azure_openai_spec_curl_args, dingllm.handle_openai_spec_data)
      end

      local function azure_openai_help()
        dingllm.invoke_llm_and_stream_into_editor({
          url_name = 'AZURE_OPENAI_API_BACKEND',
          api_key_name = 'AZURE_OPENAI_API_KEY',
          system_prompt = helpful_prompt,
          replace = false,
        }, dingllm.make_azure_openai_spec_curl_args, dingllm.handle_openai_spec_data)
      end
      -- Keymaps
      vim.keymap.set({ 'n', 'v' }, ',l', azure_openai_replace, { desc = 'llm azure_openai' })
      vim.keymap.set({ 'n', 'v' }, ',L', azure_openai_help, { desc = 'llm azure_openai_help' })
    end,
  },
}
```

Finally, add the following to `init.lua`
```
require("lazy").setup("plugins")
```
