# FSD documentation #

An unofficial attempt at documenting the implementations of FSD used today. IVAO and VATSIM happen to be the most common ones, though there are likely others.

[The documentation can be viewed here](http://web.uvic.ca/~norrisng/fsd-doc/).



## Editing and building the site ##

Individual pages can be edited using your favourite text or Markdown editor.

The entire documentation uses [MkDocs](https://www.mkdocs.org/).

```bash
sudo apt-get install mkdocs
cd fsd-doc
mkdocs build
```

This creates a directory named `site/`. The index page is located at `site/index.html`.

To see a live preview while editing, run the following:

```bash
mkdocs serve
```

Once running, the preview can be viewed at http://localhost:8000.

The table of contents on the left is controlled by `mkdocs.yml`. To edit it, please refer to the [official MkDocs documentation](https://www.mkdocs.org/#adding-pages).



## Contributing ##

Feel free to fork and create a pull request!