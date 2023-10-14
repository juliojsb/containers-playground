# Containers Playground

Para trabajar directamente en la terminal utilizaremos VIM.

Configuración VIM optimizada para YAML:

```yaml
syntax on
set tabstop=2 softtabstop=2 shiftwidth=2
set expandtab
set number ruler
set autoindent smartindent
syntax on
filetype plugin indent on

" Force saving files that require root permission 
cnoremap w!! w !sudo tee > /dev/null %
```

Otros editores de texto:

* [SublimeText](https://www.sublimetext.com/)
* [Gedit](https://wiki.gnome.org/Apps/Gedit)
* [Notepad++](https://notepad-plus-plus.org/)
