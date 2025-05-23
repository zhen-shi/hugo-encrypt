## hugo-encrypt

`hugo-encrypt` is a golang port of [Hugo Encryptor](https://github.com/Li4n0/hugo_encryptor)

`hugo-encrypt` is a tool that encrpyts content in your [Hugo](https://gohugo.io) posts. It uses AES-256-GCM to encrypt the contents of your posts, and inserts the necessary javascript code into the encrypted posts that decrypts the content after the correct passphrase has been entered.

P.S. due to the usage of WebCrypto API, hugo-encrypt only support secure content (ie. HTTPS), so unsecure content cannot call `crypto.subtle` to decrypt post.

1. [Installation and Usage](#installation-and-usage)

2. [Configuration](#configuration)

3. [Using the `{{% hugo-encrypt %}}` tag in your blog posts](#using-the--hugo-encrypt--tag-in-your-blog-posts)

### Installation and Usage

    $ export HUGO_BLOG=/path/to/hugo/blog

Place `shortcodes/hugo-encrypt.html` in the shortcode directory of your blog:

    $ mkdir -p $HUGO_BLOG/layouts/shortcodes
    $ cp shortcodes/hugo-encrypt.html $HUGO_BLOG/layouts/shortcodes

Merge i18n translation files or add it to an existing language file. Remember to set language in your [configuration](#configuration).
    
    $ cat i18n/en.toml >> $HUGO_BLOG/i18n/en-us.toml
    $ cp -r i18n $HUGO_BLOG

**Option A:** Use prebuilt binary

- **For local usage:** Download [hugo-encrypt](https://github.com/Izumiko/hugo-encrypt/releases/latest) and run it

		$ # If not in $PATH, add it first
		$ export PATH=/path/to/hugo-encrypt:$PATH
		$
		$ hugo
		$ hugo-encrypt -sitePath $HUGO_BLOG/public

- **For CI/CD usage (Vercel etc.):** Customize install command and build command

		Install command: curl -L -o hugo-encrypt "https://github.com/Izumiko/hugo-encrypt/releases/download/v0.1.0/hugo-encrypt-linux-64" && chmod 755 hugo-encrypt
		
		Build command: hugo -D --gc && ./hugo-encrypt


**Option B:** Build it

- Requirements: go 1.11+

- **Step 1:** Install `hugo-encrypt`.

	(Optional) set the BINDIR to specify a custom install location (default: /usr/local/bin)

		$ # export BINDIR=$HUGO_BLOG
		$
		$ # Then build and install
		$
		$ make
		$ make install

- **Step 2:** Use hugo-encrypt to encrypt content

        $ # If not in $PATH, add it first
        $ export PATH=/path/to/hugo-encrypt:$PATH
        $
        $ hugo
        $ hugo-encrypt -sitePath $HUGO_BLOG/public

**Option C:** Use docker

- Requirements: docker

        $ hugo
        $ docker run -it --rm \
            -v $HUGO_BLOG/public:/data/site \
            chaosbunker/hugo-encrypt hugo-encrypt -sitePath /data/site`

After generating the site with `hugo` and running `hugo-encrypt` all the private posts in your `public` directory are encrypted and the site can be published.

### Configuration

#### Setting a global password

```toml
[params.HugoEncrypt]
    Password = "yourpassword"
```

#### Password storage

`hugo-encrypt` uses _localStorage_ by default. This means the passphrase is permanently stored in the browser. By adding the `hugoEncrypt.Storage` param in your blog's config file you can set the storage method to _sessionStorage_.

```toml
[params.HugoEncrypt]
    Storage = "session" # or "local"
```

**localStorage**:

Once a visitor entered the correct passphrase the authorization status will not expire. The visitor can read the article until the passphrase has been changed or the browser cache is cleared.

**sessionStorage**:

Once a visitor entered the correct passphrase the authorization will expire after the browser is closed.

### Using the `{{% hugo-encrypt %}}` tag in your blog posts

If no password is specified in the shortcode, the password set in your config file will be used. If no password is set at all, generation of html with `hugo` fails.


```markdown
---
title: "This Is An Encrypted Post"
---

This content is visible to anyone.

{{% hugo-encrypt "postspecificpassword" %}}

This content will be encrypted!

{{% /hugo-encrypt %}}
```

#### Language

Use i18n to display content generated by `hugo-Encrypt` in the language of your choice by setting the following param in your config file. Make sure to set the Param `DefaultContentLanguage` and add the corresponding language file to the i18n folder.

```toml
[params]
    DefaultContentLanguage = "en-us"
```

### Attention

- Remember to keep the source files of your encrypted posts private. Never push your blog directory into a public repository as the encryption happens after generating the html files with `hugo`.

- Remember to run `hugo-encrypt` after generating your site with `hugo` to encrypt the posts you want to be passphrase-protected. You might want to use a shell script instead of `hugo` to take care of both steps:

```bash
#!/bin/bash

hugo --cleanDestinationDir
hugo-encrypt
```

- **To prevent the accidental leaking of sensitive data, pay attention to whether the theme you used will output a summary or full text on the article list page or RSS page. If so, please make sure that hugo-encrypt.rss.xml is added with an empty file.**
