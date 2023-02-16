## Commands

* `mkdocs new [dir-name]` - Create a new project.
* `mkdocs serve` - Start the live-reloading docs server.
* `mkdocs build` - Build the documentation site.
* `mkdocs -h` - Print help message and exit.

## Project layout

```sh
mkdocs.yml    # The configuration file.
docs/
    index.md  # The documentation homepage.
    ...       # Other markdown pages, images and other files.
```







**内容选项卡**





=== "C"
    ``` c linenums="1"
    #include <stdio.h>
    int main(void) {
      printf("Hello world!\n");
      return 0;
    }
    ```
=== "C++"
    ``` c++ linenums="1"
    #include <iostream>
    int main(void) {
      std::cout << "Hello world!" << std::endl;
      return 0;
    }
    ```





??? note

    Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla et euismod
    nulla. Curabitur feugiat, tortor non consequat finibus, justo purus auctor
    massa, nec semper lorem quam in massa.

??? a

    aa bb cc ee
    eeeeee





??? cc

    hello word