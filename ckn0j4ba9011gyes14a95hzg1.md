---
title: "Using SSH to connect to GitHub"
datePublished: Mon Dec 03 2018 00:00:00 GMT+0000 (Coordinated Universal Time)
cuid: ckn0j4ba9011gyes14a95hzg1
slug: using-ssh-to-connect-to-github
cover: https://cdn.hashnode.com/res/hashnode/image/upload/v1617381387799/XbQDqZzs1.jpeg
tags: github, ssh

---


To access GitHub more easily, I've generated an SSH key to secure the connection. To be honest, I've never created an SSH key before and it was a bit puzzling to get it to work.

From a friend, I heard that I could use PuttyGen to easily create an SSH key. After creating a few, I noticed that there were problems and that if I registered the keys I could not connect to [GitHub](https://github.com/). It kept giving me connection issues saying that I'm not permitted to access my repositories with the current key.

That meant that I had to fall back to the command line. Now I've grown fonder of the command line in the past year, but I still prefer a UI to get things done. It just resonates more with my way of working. I find that I learn a tool more easily if it has a UI component. Nonetheless, the CLI is a very powerful tool to be able to use.

After a bit of reading how to generate a key, I came up with the following command: `ssh-keygen.exe -b 4096 -t rsa -N 'ChooseStrongP4ssw0rd!' -C home -f ~\.ssh\home.ppk`.

The `ssh-keygen.exe` is the application I'm going to run. The `-b 4096` defines the number of bits the key will have. A higher number means a stronger and harder to guess key.  I've read that the minimum should be 2048 bits, so 4096 bits is better than average. The argument `-t rsa` defines the encryption algorithm to be used. There are a number of options, but [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)) is a sturdy option. To identify the key more easily, I specify a comment via `-C home`. I could use a phrase, but then I would have to surround the text in quotes: `-C 'More text as comment'`.

Setting a passphrase with `-N 'ChooseStrongP4ssw0rd!'` ensures that if the SSH key ever falls into wrong hands, it can't be used to get to my GitHub repositories without this phrase. The disadvantage is that I have to enter the key every time. I will show how to get around this later in this post. If I don't specify a password, the app will ask me for one, but you can leave it blank to create a key without a passphrase. I recommend to specify a passphrase. I generated one with my password manager [1Password](https://1password.com/) so it's unique. [The only secure password is one you can't remember.](https://www.troyhunt.com/only-secure-password-is-one-you-cant/)

Lastly, and this is a mistake on my end, I supplied a name through the `-f ~\.ssh\home.ppk`. The mistake is that if I didn't supply a name, it automatically uses the name `~\.ssh\id_rsa.ppk` which is a default name that will be picked up automatically by a lot of applications. Now I have to add a file in the `~\.ssh\` folder to let all applications know which file contains the key I want to use. Luckily this file is easily created: name it `~\.ssh\config` (don't use an extension) and put the following lines in there:

```
Host gitlab.com
IdentityFile ~/.ssh/home.ppk
```

Now, I don't want to type/copy my password every time I need to pull or push something from GitHub. My first thought was to use Pageant, the Putty ssh key manager. Unfortunately, when I loaded the generated `home.ppk` key, I get the following error.

![pageant-unsupported-key](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381384705/Mh11m_thw.jpeg)

That's when I learned that [OpenSSH](https://www.openssh.com/), who created the application `ssh-keygen.exe`, also has an `OpenSSH Authentication Agent` that can memorise ssh keys. By default, it is disabled on windows. In the `Services` program, find the `OpenSSH Authentication Agent` and enable it. Because I don't need this every day on my personal PC, I set this to manual startup. On a development machine, I would let this start up automatically with every reboot.

![services-openssh-agent](https://cdn.hashnode.com/res/hashnode/image/upload/v1617381386186/Lrw_EcItO.jpeg)

Now that the authentication agent is running, I can add the key with the following command: `ssh-add.exe ~\.ssh\home.ppk`. I fill in the password one last time to register the key and now I'm able to just use the key on my PC without entering the password every time.

By default, `git` makes use of its own SSH client (I think), because I had to set the environment variable `GIT_SSH_COMMAND` to `'C:\\Windows\\System32\\OpenSSH\\ssh.exe -T'` to use the correct SSH agent. Use the command `[Environment]::SetEnvironmentVariable("GIT_SSH_COMMAND", "C:\\Windows\\System32\\OpenSSH\\ssh.exe -T", "Machine")` to set the environment variable to the machine level so after a reboot, the variable is still set.

The last step is to register the public key with GitHub so I have permissions to use the key to update repositories via the command line. On the [GibHub Keys Settings](https://github.com/settings/keys) page, I can register a public key. The public key is generated together with the private key and stored in a file ending with the .pub extension. So my public key is generated in the file `~\.ssh\home.ppk.pub`. Open the file with Notepad or another text editor, copy the value from the file into the GitHub text box and give it a descriptive name.

Enjoy SSH'ing into GitHub, or any other service or computer that supports SSH, with ease.
