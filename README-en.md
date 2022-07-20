# IDAVSCode

Debugging IDAPython in VSCode!

Referenced from [idacode](https://github.com/ioncodes/idacode)

<p align="center">
  [<a href="README.md">中文</a>]
  [English]
</p>

## Features

![](idavscode/image/demo.webp)

## Requirements


1. `python >= 3.10` # Theoretically, replacing the `match case` statement should make 3.x fully compatible.
2. `python -m pip install debugpy tornado`.
3. [Python extensions for VSCode](https://marketplace.visualstudio.com/items?itemName=ms-python.python)


## Install

1. download the IDA plugin and extract it to the Plugin folder.
2. Install the [Vscode extension](https://marketplace.visualstudio.com/items?itemName=Cirn09.idavscode)

## launch.json

````json
{
    "name": "IDAPython debug",
    "type": "idapython",
    "request": "launch",
    "program": "${file}",
    "host": "localhost", // control hostname
                                                        // Same as the IDA side
    "port": 5677, // control port
                                                        // Same as IDA side
    "pythonPath": "${command:python.interpreterPath}", // python path (used by IDA)
                                                        // If VSCode and IDA use the same version, don't bother with this field.
    "logFile": "", // debug log file
    "debugConfig": { // Python extended debug configuration
                                                        // Debug configuration passed to Vscode Python extensions
        "type": "python",
        "request": "attach",
        "justMyCode": true,
        "connect". {
            // "host": "localhost", // optional, default use upper "host" as control host if not filled.
                                                        // optional, default use upper level "host" if not filled
            "port": 5678 // debug port
        },
        "cwd": "${workspaceFolder}"
    }
}
```

## Principle

The plug-in on the IDA side starts two services, a control service and a debug service ([debugpy](https://github.com/microsoft/debugpy)).

The VSCode side expansion intercepts debugging tasks and sends debugging context information to the IDA control service, which is responsible for starting the debugging service and running the target script according to the context. When ready, the VSCode extension replaces the debug task with a Python Remote Attach task. When the script finishes running, the VSCode extension actively stops debugging.

## Known defects and pending

- [x] The debugged script does not debug again properly after an exception is triggered by the debugged script.
    It should be related to the IDAPython thread, where the control and debug services are running on a sub-thread, and the debugged script is running on the main thread via `ida_kernwin.execute_sync`. Add `ida_kernwin.refresh_idaview_anyway()` and it seems to work.

- [ ] Debug service cannot be terminated: debug service uses [debugpy](https://github.com/microsoft/debugpy), debugpy currently only provides an interface to start the service, not to stop it ([related issue](https://github.com/ microsoft/debugpy/issues/870)). So debugpy will keep occupying the debug port after the debug service is started.

    Temporary solution: you can restart IDA manually, or end the Python subprocess of IDA (this subprocess is started by debugpy, IDAPython uses Python.dll and does not start a separate Python process)

- [ ] (Possible) Python debug settings do not work: The VSCode plugin intercepts replacement debug tasks using `vscode.DebugConfigurationProvider { resolveDebugConfiguration, resolveDebugConfigurationWithSubstitutedVariables }` two interfaces, where `resolveDebugConfiguration` is used to supplement the debug configuration, and ` resolveDebugConfigurationWithSubstitutedVariables` to replace debugging tasks.

    In practice, the effect of replacing tasks with `resolveDebugConfigurationWithSubstitutedVariables` is not exactly the same as calling `vscode.debug.startDebugging`. Grabbing the package reveals that the former creates a debug session with fewer options, presumably because this approach bypasses some of the `launch.json -> session config` processes.

- [ ] `pythonPath` in `launch.json` is automatically corrected to `python`, not sure if it's just a problem on my end or if it's common.
- [ ] Icon
- [ ] Test
- [ ] Slice and dice the process to provide more granular commands