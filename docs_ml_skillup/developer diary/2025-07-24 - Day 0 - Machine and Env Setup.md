Welcome to Day 0 of the rest of this journey. After setting up this repo, I made a quick list of the courses that I had signed up on Coursera and LinkedIn Learning. There are other sources as well, but this is to make a start. I'll soon start mapping courses to the skills present in [Initial Plan](Initial%20Plan.md).

# System Setup
- I'm starting this project with my trusty Macbook Air laptop which had replaced my Windows laptop 2-3 years ago as my go to laptop but I had been shying away from switching to Mac for development completely.
- But given that my Windows laptop is now breathing it's last, I think it is time I made the switch. Plus, my work laptop might get switched from Windows to Mac as well, so consider this me preparing for future.
## Homebrew and Xcode
- The first thing was to understand the Mac Dev landscape. Enter Homebrew, THE package manager for MacOS. It also needs Xcode tools which made me sus since I thought Xcode is for Mac/iOS development but turns out `Xcode Command Line Tools` is used by Homebrew so I'll allow it.
	- I think Homebrew is already installed. I updated it's version `brew update` and some dependencies were outdated which I updated using `brew upgrade`. 
	- Btw you can find the outdated packages, (i think homebrew calls it 'formulae') by doing `brew outdated`
## Python Setup 
### pyenv
- Next we install `pyenv` for python version management.
```bash
brew install pyenv
```
- **Configuring shell:** Add the following to your `~/.zshrc` (open by doing `open -e ~/.zshrc`)
```bash
    # pyenv setup
    export PYENV_ROOT="$HOME/.pyenv"
    export PATH="$PYENV_ROOT/bin:$PATH"
    eval "$(pyenv init --path)"
    eval "$(pyenv init -)"
    eval "$(pyenv virtualenv-init -)" # Important for virtualenv integration, which Poetry uses
```
- **Restart your terminal** or run `source ~/.zshrc`.
- Verify: `pyenv --version`
### Installing Python
- As mentioned earlier, I seem to have already installed Python using Anaconda, but I want to avoid that this time. I considered deleting Anaconda completely but I wonder if I might have some stray Jupyter notebook or something that relies on it, and frankly, I don't want to waste time fixing it later. So as long as it doesn't come in my way, I'll let the existing py3.9 though anaconda stay and install 3.10.12 for now using `pyenv`
```bash
pyenv install 3.10.12
pyenv global 3.10.12
```

### Installing Poetry
```bash
pip install poetry
```
- Additionally, Gemini tells me that I need to add Poetry to PATH but there is a simpler way to let Poetry manage virtual environments inside the project directory. I like that idea so will run the following:
```bash
poetry config virtualenvs.in-project true
```

## Tool Setup
### VS-Code
```zsh
brew install --cask visual-studio-code
```

### Docker
```zsh
brew install --cask docker

# once done, run to verify installation
docker run hello-world
```
- Installation involved signing up to docker as well. Docker is one of those things that I'm not too comfortable with, so I thought why not. Hope it was not a mistake.

### Kubernetes Command Line Tool
```zsh
brew install kubectl
```

```zsh
# Verifying Installation
kubectl version --client
```


### Minikube
```zsh
brew install minikube
```

- To start/stop minikube:

```zsh
minikube start
minikube stop
```
