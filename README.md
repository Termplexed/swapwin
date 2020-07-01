# swapwin

Locate and raise (g)vim holding swap file. 

Uses various hacks to try to find the vim instance holding the file. For GVim 
it is fairly OK as the gui usually has its own PID. For vim it is harder and 
more error-prone.

* Linux like

## Depends on

| Required | Either or | Optional |
| -- | -- | -- |
| bash, wmctrl, awk, ps, find | lsof \|\| xxd | xprop, xwininfo, luck |


## Usage


    swapwin [[option] <PID>] | [[option] <FILE|SWP>]


| command |  | | | |
| --- | ---: | ---: | ---: | ---: |
| `swapwin j .some_file.c.swp` |  | swap | → pid | → window ◑ |
| `swapwin j some_file.c` | file | → swap | → pid | → window  ◑ |
| `swapwin j 1234` | | | pid | → window ◑  |

– *If window is not found from pid, which happens on vim, we go:*

◑  pid's open files → window titles with file names → window

*If one specify file by name it is required that the swap file reside in same 
directory.*

---

| option | *Intended result* |
| - | :- |
| `j`| Jump to Workspace and raise window. (Default) |
| `g`| Fetch Window if on other workspace and raise window. |
| `p`| Only print information. |
|  |  |
| `L`| It uses xxd and /proc/ by default. With -L it uses lsof. |

<section>
<details>
<summary>shell-help</summary>

```
Usage: swapwin [[opt] <PID>] | [[opt] <FILE|SWP>]

Find VIM window holding swap file

OPTIONS:
   j   : Jump to window. (Default)
   g   : Get window.  (E.g. from other workspace.)
   p   : Only print.  (With some extra info.)
   L   : Use lsof.    (Default xxd + /proc/)
   s   : Alias for j. (switch)
   i   : Alias for p. (information)
   h   : This help.

  <PID>: Process ID.
  <SWP>: Read PID from Vim swap file. (Using xxd or lsof)
 <FILE>: Try to locate swap from file-name and work from there.
```

</details>
</section>

## Details

<section>
<details>
<summary>Read arguments from user, if</summary>

**Swap file + xxd / proc:**

* Read PID from swap file
* Loop trough windows by wmctrl and if PID has a window, raise and end, else
	* Loop file-descriptors linking to swp files under /proc/PID/fd
	* Loop trough windows by wmctrl and check if file is part of the title
	  of any windows.
	* If found, raise and end.

**Swap file + lsof**

* Get PID by lsof on swap file
* Loop trough windows by wmctrl and if PID has a window, raise and end, else
	* Loop open files ending with .sw?, fetched by lsof
	* Loop trough windows by wmctrl and check if file is part of the title
	  of any windows.
	* If found, raise and end.

**File**
Try to locate swap file in same direcory as target, if found countinue as 
above.

**PID**
As above but without looking at swap file.

</details>
</section>

<sub>As some of the messages in the script show, this is not the most serious project.</sub>

## Todo

Could be smarter on filtering duplicate windows. As of now it list all sessions that have
a file matching with *one* of the files in the window we are after. So if you are after

* foo.txt

And the window holding it also have *bar.txt* and *baz.txt*, it matches all sessions that have
files that are either *foo.txt*, *bar.txt* or *baz.txt*.

Easy fix I believe, but have not been big an issue.
