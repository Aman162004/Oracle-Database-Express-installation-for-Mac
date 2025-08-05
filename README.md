# Oracle-Database-Express-installation-for-Mac
---
## Install home brew
step 1: echo 'eval "$(/opt/homebrew/bin/brew shellenv)"' >> ~/.zprofile

step 2: eval "$(/opt/homebrew/bin/brew shellenv)"

step 3: brew --version

Output:
Homebrew 4.x.x
Homebrew/homebrew-core (git revision ...)

---

## Install Colima and Docker CLI(Not Docker Desktop)
step 1: brew install colima

step 2: brew install docker

---

## Start Colima in x86_64 mode
step 1: brew install qemu

step 2: colima start --memory 4 --arch x86_64


