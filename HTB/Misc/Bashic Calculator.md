# Bashic Calculator

######  Misc, Easy - x1foideo


## Challenge Description

> I've made the coolest calculator. It's pretty simple, I don't need to parse the input and take care of execution order, bash does it for me!I've also made sure to remove characters like $ or \` to not allow code execution, that will surely be enough.


## Initial Recon 

#### Goal
Analyzing the *main.go* file it's possible to notice at the end of it:

```go
command := "echo $((" + op + "))"
```

However looking through the internet, we find bad news, since the **$((expression))** is an *Arithmetic Expansion*, meaning that is only able to solve "Calculations".
The goal here would be to replace the *Expression* with something able to execute some code, something like

```bash
echo $(( $(command) ))
```

#### Filters
Unfortunately it's not possible to use lots of symbols, for they are filtered

```go
firewall := []string{" ", "`", "$", "&", "|", ";", ">"}
```


## Solution

A good idea is to transform the *Arithmetic Expansion* to a *Command Substitution*, **$(expression)** simply getting rid of a bracket.
Searching for something more on these expansion I came across a quite simple but exhaustive explanation:

```
$(()) -- Good
$( () ) -- Bad
$(() ) -- Bad
$( ()) -- Bad
```

We cannot change the first two parenthesis, but neither the last two. However, *Round Brackets* are not whitelisted.

```bash
echo $((whoami) ) #))
```

**Whitespaces are Filtered, TABS are not fortunately** 

At this point the idea now resulting is:

```bash
echo $((command)    )    #))
```

So the payload is going to be:

```bash
whoami)    )    #
```

Check where the flag is and send the corrected payload

Get the Flag


#artihmeticexpansion #commandsubstitution #filter #whitespace #tab