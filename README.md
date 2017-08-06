# b0tchsec.com

## Local build
I recently encountered issues building and running the site on the latest version of Jekyll.  I was able to workaround this by rolling back to an old version of Jekyll.

```
sudo gem install jekyll --version 3.2.1
jekyll _3.2.1_ build
jekyll _3.2.1_ serve
```
