# [b0tchsec.com](https://b0tchsec.com)

## Local build
The site is currently not compatible with Jekyll >=3.5.0.  So I have added a Gemfile to help with installation of the correct version.

```
bundle install
jekyll build
jekyll serve
```

If you encounter the following error when building/serving the site:
```
Liquid Exception: undefined method `scan' for #<Liquid::VariableLookup:0x0055555872ca10> in sitemap.xml
```
Then you likely have multiple versions of jekyll installed.  Either uninstall the jekyll versions >=3.5.0 with `sudo gem uninstall jekyll` or specify the desired version at run-time.

```
jekyll _3.4.5_ build
jekyll _3.4.5_ serve
```
