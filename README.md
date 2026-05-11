# humorless.github.io

How to run the local blog server
```
lein ring server-headless
```

Or more aggresive
```
tmux new-session -d -s lein-ring 'lein ring server-headless'
```
