{:title "Clojure development environment by Vagrant"
 :layout :post
 :date "2019-05-13"
 :tags [ "DevOps", "vagrant", "vim-fireplace", "ansible"]}

## TL;DR
If you want to have a portable Clojure development environment and you use Vagrant, vim-fireplace, you may consider to try my [Vagrantfile](https://github.com/humorless/dotfiles). 

```
git clone https://github.com/humorless/dotfiles
cd dotfiles
vagrant up
```
## Certain part of vagrantfile you may need to remove.
```ruby
if Vagrant.has_plugin?("vagrant-timezone")
  config.timezone.value = "Asia/Taipei"
end
```

## The beginning of this repo

Several years before, I created a github repo called `dotfiles`, which is used to record my vimrc file. Later, every time when I changed my job, I modified my favorite vim plugin. I modified my vim plugin collection so many times. Sometimes, I installed certain vim cool plugin, but after a while, I totally forgot how to use it. There are not too many vim plugins in this dotfiles, because I am not a vim `l33t hax0r`.

## development and deployment

I have had a job that I needed to work at AWS cloud9 environment. Some of my jobs required me to install totally new development environment. Recently, I needed to deploy Clojure enviroment on production system, so I learned a little `ansible` and I used ansible to install java8.

One day, I found that vagrant can use ansible to do provisioning, so I combined them together.

## Some nice tools I cannot live without

`nvm` is important to me because I usually need to change node version. `autojump` is also important. 
