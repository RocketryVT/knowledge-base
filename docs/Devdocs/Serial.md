## Hello, is Anyone Home?
Rather unfortunately for all of us, most small embedded systems do not have ears, and thus have zero appreciation for the beautiful poetry you try to speak to them - or code you try to upload to them verbally, for that matter. Rather, any information that needs to be communicated to and from an embedded device typically has to happen over a cable; inside that cable are a couple wires that make up a **serial bus**, which is the medium over which all of our encoded information will actually be transmitted. Luckily, you have many of these buses available to you on your laptop, or any computer - USB literally stands for **U**niversal **S**erial **B**us, and I'd put money on you having at least one USB port at your disposal on any device you'll be writing code on. 

Once you have our code running on a given device, you'll need to run a program to tell your computer to listen with its digital ears on the given serial port that you have the device plugged into. Rather inconveniently, this program is different on Windows than it is on Linux/Unix; you *technically* can expose your underlying USB ports to WSL2 and run the same program (`minicom`) in a WSL terminal as you can in Linux/Unix, but the process for doing so is relatively waffley and can be unreliable, so we recommend using `PuTTY`, a standalone program, instead. The process for installing these programs is included in the installation guide for their respective operating systems, and thus will not be repeated here - but thankfully, once they are installed and started up, their usage inside the program is exactly the same regardless of which OS you're using. The rest of this doc will specifically be about the usage of the [serial tool](https://github.com/RocketryVT/active-drag-system/blob/main/src/serial.cpp) included in our shared codebase, and how to write additional commands into it and modify its usage for testing.

## A Terminal on a Rocket
Once you connect a board and boot up your respective serial program, you should see something like the below - if not, try typing and entering `info` once or twice, and it should pop up; you likely opened the program after the below message was sent, and thus the program wasn't running when that data was received.

```bash
#####################################################################################
#                                                           =                       #
#                                                          =                        #
#                                                         =                         #
#                                                                                   #
#                                                      #                            #
#                                                     #                             #
#                                    ==========      ##                             #
#                             ==========        ===###                              #
#                         =======                 ### =====                         #
#                      ======                    ###     =====                      #
#                   ======                      ###         =====                   #
#                 ======                       ###            =====                 #
#               ======                        ###               =====               #
#   #########  =====                        ####                  ====  ########    #
#   ##########                             #####                       #########    #
#   ##################################### ##### ################################    #
#   #################################### #####  ################################    #
#   #############   ##   ##  ##   ##   ######  ##   ##  ###  ##   ##############    #
#   #############   ##   ##  ##   ##  ######   ##   ##  ###  ##   ##############    #
#   #############   ##   ##  ##   ##  #####    ##   ##  ###  ##   ##############    #
#   ###############################  ###### ####################################    #
#   ##############################  #######     ################################    #
#   ###################            #######                  ####################    #
#   #############                 ########                        ##############    #
#   ############                 #########                        ##############    #
#   ############                ###########                       ##############    #
#   ############               #############                      ##############    #
#   ############             ##################                   ##############    #
#   ############          #########################               ##############    #
#                                                                                   #
#####################################################################################
#####################################################################################
#                          Rocketry at Virginia Tech                                #
#                         Executeable: your_executable.uf2                          #
#####################################################################################
```

This is the raw output of the `info` command of the serial tool, which is configured to run on startup (or when you just typed it in and ran it manually). All of what you see in your serial terminal came off of your board directly - this terminal emulates one that gives you access to "inside" the board you're working on, and its processes.

The tradeoff to this is that, while there is actually an operating system running on our boards coordinating things (FreeRTOS), there is no *actual* terminal - just a program listening for individual serial characters, and doing certain things if the strings match up. This means that every command that you can run has to be manually added, though this isn't as hard as it sounds. Let's look at the list of commands available to you, by running the `help` command.

```shell
# help
Commands:
		help
		info
		clear
		top
		reset
		read
		write
		erase
		show
		deploy
		kalman
# 
```

Now, let's break these down one by looking at where they're defined. There are five "default" commands, defined within the `serial.cpp` file and its accompanying `.h` header; they can be found right [here](https://github.com/RocketryVT/active-drag-system/blob/320d5c7a5b3e29f38a31e0a5399f47f008e036fd/include/serial.hpp#L68):
```cpp
#define NUM_BASE_CMDS 5
const command_t base_commands[] = { {.name = "help",
                                     .len = 4,
                                     .function = &help_cmd_func},
                                    {.name = "info",
                                     .len = 4,
                                     .function = &info_cmd_func},
                                    {.name = "clear",
                                     .len = 5,
                                     .function = &clear_cmd_func},
                                    {.name = "top",
                                     .len = 3,
                                     .function = &top_cmd_func},
                                    {.name = "reset",
                                     .len = 5,
                                     .function = &reset_cmd_func}};
```

This is the default format for adding commands. Of the five defined here, they all do very simple housekeeping tasks:

- `help` does exactly what we just saw, and lists out all available terminal commands
- `info` prints the Rocketry@VT logo splash, and lists the currently running executable.
- `clear` clears the output of the terminal, so you have a "blank slate" to run more commands.
- `top` does exactly what it does on Linux, and prints the top "tasks" running on the stack currently and the resources that they're using. Note that usually, these tasks will be named exactly what you define them as in code:
- `reset` does exactly what it says on the tin, and resets and reboots the processor. **This command seems to be a bit buggy currently, so be wary when using it.**
```shell
# top
Tmr Svc         7               <1%
serial          450262          99%
IDLE1           227616          50%
IDLE0           227345          50%
logging         1               <1%
rocket_task     1               <1%
kalman_task     206             <1%
heartbeat       0               <1%
launch_event_ha 87              <1%

        Available Heap Space In Bytes:  23576
Size Of Largest Free Block In Bytes:    23576
Number Of Successful Allocations:       21
#
```

The rest of the commands are defined in `active_drag_system.cpp`, right [towards the top](https://github.com/RocketryVT/active-drag-system/blob/320d5c7a5b3e29f38a31e0a5399f47f008e036fd/src/active_drag_system.cpp#L84). These are more purpose-built tasks added in the main script as necessary, so let's go through the ones that it adds.

- `read` literally reads the contents of the flash memory into the terminal for local logging. This gets formatted according to `log_format.hpp`, so be mindful about the format of the data that you're saving.
- `write` does the exact opposite, and writes a single log entry to the flash for debugging.
- `erase`, well, *erases* the full contents of the flash memory aside from the currently running code (i.e., any stored data from flights or testing). **There is no confirmation dialog for this, it just runs - so make sure you've saved the data onboard beforehand if necessary.**
- `show` will print out all currently logged packets to the terminal, in real time - so get ready for a *lot* of output. To turn it off, simply run `show` again - the individual characters may scroll up in the list as the output runs, but it'll still parse what you're typing normally.
- `deploy` will run a quick deployment sequence on the servo GPIO pin to confirm that it's working normally; if no servo is plugged in on your board, no visible feedback will be produced.
- `kalman` is a testing command that will print the output from the onboard kalman filter. If you don't have the `DEBUG=1` flag set in your compilation, then this will print the headers of the data outputs, and then stop.

Now, let's get to the fun part - adding *new* commands.

## Adding Serial Commands
To add commands to the serial tool, you've already seen all of what you need to. New commands should be added to the "main" file that gets turned into your executable, i.e. for this case, `active_drag_system.cpp`. There are three things that need to be added for the command to be able to be run:

1. The variable that stores the number of commands must be incremented.
2. The command must be added to the list, and given a name, a length (in string characters), and a method name that will be run when it is executed.
3. A method with the name as configured must be defined; *this is where your actual code will go.*

So, let's add a very simple command to demonstrate this - we'll add a `hello` command, which prints "Hello, World!" to the serial terminal.

1. We'll increment the [num_user_cmds](https://github.com/RocketryVT/active-drag-system/blob/320d5c7a5b3e29f38a31e0a5399f47f008e036fd/src/active_drag_system.cpp#L93) from 6 to 7, as we're only adding one.
2. We'll add a line for the `hello` command, and link it to a method we'll call `hello_world_func`, following the naming convention used for the rest.
3. Finally, we'll define `hello_world_func()` as a function in the initial block with the rest, and then actually populate it with our code down below the rest of the method definitions.

```cpp
static void read_cmd_func();
static void write_cmd_func();
static void erase_cmd_func();
static void show_cmd_func();
static void deploy_cmd_func();
static void kalman_cmd_func();
static void hello_world_func();

const char* executeable_name = "active-drag-system.uf2";
const size_t num_user_cmds = 7;
const command_t user_commands[] = { {.name = "read",
                                     .len = 4,
                                     .function = &read_cmd_func},
                                    {.name = "write",
                                     .len = 5,
                                     .function = &write_cmd_func},
                                    {.name = "erase",
                                     .len = 5,
                                     .function = &erase_cmd_func},
                                    {.name = "show",
                                     .len = 4,
                                     .function = &show_cmd_func},
                                    {.name = "deploy",
                                     .len = 6,
                                     .function = &deploy_cmd_func},
                                    {.name = "kalman",
                                     .len = 6,
                                     .function = &kalman_cmd_func},
                                    {.name = "hello",
                                     .len = 5,
                                     .function = &hello_world_func} 
};

//Down below the rest of method definitions:
static void hello_world_func() {
	printf("Hello, World!\n");
}
```

And if we compile and run this code, our new command will show up in the list, and can be executed!

```shell
# help
Commands:
        help
        info
        clear
        top
        reset
        read
        write
        erase
        show
        deploy
        kalman
        hello
# hello
Hello, World!
#
```



