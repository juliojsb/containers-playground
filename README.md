# Containers Playground

## Configuración VIM optimizada para YAML

```yaml
set tabstop=2 softtabstop=2 shiftwidth=2
set expandtab
set number ruler
set autoindent smartindent
syntax enable
filetype plugin indent on

" Force saving files that require root permission 
cnoremap w!! w !sudo tee > /dev/null %
```
