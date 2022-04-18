# Welcome to [My Blog](https://blog.yikang.pub/)

I use [hugo](https://github.com/gohugoio/hugo) to build my blog. And I use this [theme](https://github.com/kang8/hugo-theme) to customize my theme.

### How to build this blog in your computer

1. install hugo

    For MacOS

    ```bash
    brew install hugo

    hugo version # check hugo installed
    ```

    Other operating systems, see [hugo install document](https://gohugo.io/getting-started/installing/)

2. download this repo

    ```bash
    # https
    git clone --recurse-submodules https://github.com/kang8/kang8.github.io.git
    # ssh
    git clone --recurse-submodules git@github.com:kang8/kang8.github.io.git
    ```

3. build blog

    ```bash
    hugo server -D
    ```
