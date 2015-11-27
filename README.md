# Persist

[![Build Status](https://travis-ci.org/eschnett/Persist.jl.svg?branch=master)](https://travis-ci.org/eschnett/Persist.jl)
[![Build status](https://ci.appveyor.com/api/projects/status/yxffg2v2tb90vnf2?svg=true)](https://ci.appveyor.com/project/eschnett/persist-jl)
[![codecov.io](https://codecov.io/github/eschnett/Persist.jl/coverage.svg?branch=master)](https://codecov.io/github/eschnett/Persist.jl?branch=master)

Persistent jobs for Julia.

## What does it do?

The package `Persist` allows running jobs independent of the Julia shell. The jobs are run in the background, either on the local machine or via Slurm, and are not interrupted when the Julia shell exits. This is a convenient and safe way to start long-running calculations, without having to write a Julia script.

## Why is this a good idea?

Programming in Julia typically proceeds in two stages: First one writes some exploratory code in the shell (or via Jupyter). Later, when the code becomes more sophisticated, one converts part of the code to a Julia package that is developed in an editor outside the Julia shell. One still uses the Julia shell to test the package.

As code complexity increases, so do the run times. What takes a few seconds initially turns into minutes and then hours of run time. This then makes things inconvenient:
- While one long-running command is executing, the Julia shell is blocked
- If the command is started in the background, one may accidentally overwrite or delete data that it is accessing
- If the Julia shell exists, or the network connection is lost, the background process is aborted

This package `Persist` circumvents these problems: It allows wrapping a Julia command in a shell script that is run in the background, independent of the Julia shell.

## How do I use it?

Here is an example:

```Julia
using Persist

# Define a function
function hello()
    sleep(10)   # Simulate a long-running task
    println("Hello, World!")   # Produce some output
    return [42]   # Return a value
end

# Run the function in the background
job = @persist "hello" ProcessManager hello()

# Do something else

# Check on the background job
status(job)

# Get the job's result
fetch(job)
getstdout(job)
getstderr(job)
wait(job)
cleanup(job)
```

You can also use Slurm to submit a job:

```Julia
@persist "calcpi" SlurmManager (wait 10; big(pi))

# Jobs are written to file, and can be read back in
job = readmgr("calcpi")
jobinfo(job)
println("pi = $(fetch(job)))
cleanup(job)
```

## How does it work?

Simple, really.

The Julia expression is serialized and written to a file. A shell script is generated that is executed in the background (or via Slurm). This script reads the expression, executes it, and serializes the result to another file. Various commands examine the status of the job. `fetch` deserializes the result once the job has finished.

## Reference

### Create a job
```Julia
mgr = @persist name manager expression
mgr = persist(function, name, manager)
```
- `name::AbstractString`: Job name
- `manager::JobManager`: Either `ProcessManager` or `SlurmManager`
- `expression::Any`, `function::Any`: Expression or function to evaluate

mgr = readmgr(name)
status(mgr)
jobinfo(mgr)
cancel(mgr)
wait(mgr)
fetch(mgr)
cleanup(mgr)
```
