I'm starting to use more snippets as I often am searching through previous code to copy and paste something.

Yasnippet is my library of choice used with Emacs.

`M-X yas-new-snippet`

This opens a new buffer with a format like so 

```
# -*- mode: snippet -*-
# name: 
# key: 
# --
```

This is pretty easy, simply type in a name for the snippet, and a keyboard shortcut that can be used (followed by tab) to invoke it.

I wanted to make a snippet that allowed me to create a form tag with an example form group div class.

```
# -*- mode: snippet -*-
# name: Form Tab
# key: ftag 
# --

<%= form_tag register_create_path, :method => :post do %>
  <div class="form-group">
    <div class="field">
      <%= label_tag :team_name %><br />
      <%= text_field_tag :team_name,'', class: 'form-control', required: true, autofocus: true %>
    </div>
  </div>
<% end %>
```

When I saved this file, I found the buffer was already set up to allow me save in `web-mode`. Presumabily this was because I ran the `yas-new-snippet` command in an html file. The snippet ended up being located here  `/home/me/.emacs.d/snippets/web-mode/form_tab`

Now I can simply run `ftag <TAB>` in a web file and my snippet is available!

Special note that when defining a snippet you can setup tab stops that allow the user to easily jump to specific points of the snippet for input. See the subject snippet definition here that allows parameters to be added.

```
# -*- mode: snippet -*-
# name: Rspec Subject
# key: sub 
# --
subject { described_class.new($0) }
```
