# Initialize repository

Check out the repository:

```
git clone [address of this repo]
```

then also check out the submodules, in this case [PicoCSS](https://picocss.com/)

```
git submodule init
git submodule update
```

this will actually pull down the PicoCSS, otherwise you will not
have the stylesheets available.

# Build and update

For building

```
hugo build
```

For starting the local webserver and also including drafts

```
hugo serve -D
```
