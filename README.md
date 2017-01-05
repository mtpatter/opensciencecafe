# opensciencecafe

Blog using the **So Simple Theme**, the followup to [**Minimal Mistakes**](http://mmistakes.github.io/minimal-mistakes/) -- by designer slash illustrator [Michael Rose](http://mademistakes.com).

To run Jekyll with Docker for editing, from the root directory, build the image:

```
docker build -t "jekyll_blog" .
```

and run a container:

```
docker run --name jekyll -d -v ${PWD}:/srv/jekyll -p 4000:4000 jekyll_blog
```

