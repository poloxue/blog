
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-01.png)

I was thinking about ways to work faster in the terminal and wondered if I could make something like tmuxifier for iTerm2. 

Let's stop thinking, start doing.

This article is about how I made this tool from scratch.

## Tmuxifier

What is Tmuxifier for? 

Tmuxifier helps organize your tmux sessions, just in case you're not familiar with it.

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-02-v1.gif)

## Getting Ready

Before jumping into making the tool, I checked if it was even possible.

### iTerm2 Works with Python

First off, I found out iTerm2 can be controlled with Python scripts. This was great because it meant I could actually build my tool. iTerm2's Python API lets you do a bunch of stuff, like changing the background, making new panes, or switching tabs. You can look up more on their [Python-API page](https://iterm2.com/python-api/).

### Setting Things Up

To get started, I made sure my iTerm2 (version 3.4.23) was ready to use the Python API, and I had Python set up on my computer. Then, I installed the library needed for iTerm2's Python API:

```bash
$ pip install iterm2
```

### Planning the Config File

I chose YAML for the config file because it's easy to read and simple. My plan was to use the YAML file to describe the layout, including windows, panes, and what commands to run. Here's an example of what a tmuxifier config looks like:

```bash
# Setting the window name and root directory
session_root "~/Projects/my-project"
window_name "my-project"

# Making a new window and splitting it into two panes
new_window "my-project"
split_h 50

# Running vim in the first pane
run_cmd "vim" 0

# Running go run main.go in the second pane
run_cmd "go run main.go" 1

# Choosing the main pane
select_pane 0
```

I turned this into a YAML config file like this:

```yaml
layout:
  window_root: "~/Projects/my-project"
  window_name: "my-project"
  panes:
    - percentage: 50
      commands:
        - "vim"
    - type: "split_h"
      commands:
        - "git status"
  main_pane: 0
```

This setup is meant to make a layout with two panes: one for `vim` and another for `go run main.go`, starting with the focus on the `vim` pane.

To work with the YAML file, I used Python's `yaml` library:

```python
import yaml

# Reading the YAML config
with open('layout.yaml', 'r') as f:
    config = yaml.safe_load(f)
layout = config["layout"]
```

Now I had a dictionary with all my layout info.

The next step was to turn this plan into actions in iTerm2, creating windows and panes, and running specific commands in those panes.

## Making the Layout Manager

I wrote a Python script, let's call it itermifier, that uses my YAML config to make panes and run commands.

### Getting Ready

Before building, I set up some basics that I'd use a lot when making the layout.

First, I got the iTerm2 app ready and connected using the iTerm2 API:

```python
async def create_layout(connection):
  app = iterm2.async_get_app(connection)

iterm2.run_until_complete(create_layout)
```

This part sets up a function called `create_layout` which is key to the whole tool. The `connection` is needed to talk to iTerm2 and get things done.

### Windows and Tabs

Then, I made sure I was working with the current window and tab:

```python
window = app.current_terminal_window
tab = window.current_tab
```

This way, I made sure I was messing with the right parts of iTerm2, like splitting panes or running commands.

I also made a list called `sessions` to keep track of sessions in the tab:

```python
# A list for sessions made ahead of time
sessions = [tab.current_session] 
```

This list keeps all the sessions for the tab, starting with the current session from `tab.current_session`.

### Splitting Panes

Next, I went through my pane plans and split them as needed:

```python
for index, pane in enumerate(layout["panes"]):
  pass
```

Here's how I did the splitting:

```python
vertical = False if pane["type"] == "split_h" else True
session = await tab.current_session.async_split_pane(vertical=vertical)

session = sessions.append(session)
```

For each pane, I used iTerm2's `async_split_pane` method to split them horizontally or vertically, depending on the plan.

### Running Commands

After making each pane, I ran the planned commands using the `

async_send_text` method:

```python
for cmd in pane.get("commands", []):
      await session.async_send_text(f"{cmd}\n")  # pyright: ignore
```

This pretty much did everything I wanted.

### Back to the Main Pane

In the end, I made sure to go back to the main pane. I had all the sessions in a list, so after everything else, I could just switch to the main pane I wanted:

```python
main_pane_index = layout["main_pane"]
await sessions[main_pane_index].async_activate()
```

And that was it, tool complete.

Here's how it looked:

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-02/2024-02-26-build-an-itermifier-03.gif)

For the full code, you can check this out: [itermifier](https://gist.github.com/poloxue/c2b1a4c839750f1c5c0cba65c74a223b).

### Wrapping Up

Through this process, I managed to make a tool that automates layout management in iTerm2. There were some challenges, but being able to set up and start my work environment with code was awesome. The only bummer is it doesn't let you adjust the size ratios, but all in all, it was a cool project to work on.
