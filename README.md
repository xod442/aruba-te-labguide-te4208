# Test Drive Documentation

## Prerequisites.

- Python >= 3.10
- pip3 install -r requirements.txt

## Folder structure

- **docs** - published guide contents in subfolders (subfolder is the navigation uri for each guide).
- **docs/_logos** - shared images across all lab guides.
- **images** - images for the readme
- The rest of the folders contain the raw markdown content for each guide

## Building a new lab guide

1. Create a new folder 
2. Create a folder call `docs` inside your new folder. This will be the location for your markdown files. Create as many or few files as needed. Each file will be a new section in the table of contents (toc).
3. Copy the `_config.yml` and `_toc.yml` files from an existing lab guide folder and edit the files appropriately. The `toc.yml` file has references to your markdown files (without the `.md` extension) located in the `[your_folder]/docs` directory.
4. When satisfied with the content created in steps `2` adn `3`, run the following command to build your content in the `jupyter` format. At the end of the command, there will be a link to view your content in a browser.

```bash
jb build [your_folder]/
...
...
...
===================================================

Finished generating HTML for book.
Your book's HTML pages are here:
    testdrive/docs/_build/html/
You can look at your book by opening this file in a browser:
    testdrive/docs/_build/html/index.html
Or paste this line directly into your browser bar:
    file:///home/pod5/test2/testdrive/docs/_build/html/index.html

===================================================
```
