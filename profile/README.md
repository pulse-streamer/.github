> [!WARNING]
> This project is a work-in-progress - already usable, but some features are still missing and breaking changes are still possible. Documentation is coming soon.

# NI Pulse Streamer

High-level Python API for scripted pulse sequence generation with [National Instruments](http://www.ni.com/) hardware.

Features:
* Python front-end provides a simple API for pulse sequence scripting - users have the full power of Python syntax; 

* Streaming approach - the waveform samples are calculated on-the-fly - allows for memory-efficient sequence storage and sampling. Only a small amount of data is stored per a pulse and only a small portion of the sequence is sampled at a time. As a result, total sequence duration is practically unlimited;

* Streaming back-end is implemented in Rust - fast, lightweight, and robust;

* Package format - the streamer can either be used directly as a standalone, or be integrated into any other control software.

See [demos](https://github.com/pulse-streamer/ni-streamer/tree/main/py_api/demo) for the example workflow.

## Installation instructions
Currently, you have to git-clone the repos and build the back-end locally (pre-built and `pip`-installable version is coming later).

1. Install Rust by following the [official instructions](https://www.rust-lang.org/learn/get-started).

2. Git-clone the required repos:  
    ```
    git clone https://github.com/pulse-streamer/base-streamer.git 
    git clone https://github.com/pulse-streamer/ni-streamer.git
   ```
    
3. In terminal, navigate to `/ni-streamer/backend` directory and run  
    ```
    cargo build --release
   ```  
    to compile the Rust backend. Then run  
    ```
    maturin develop --release
    ```  
    to install `nistreamer_backend` into your Python environment. You may need to activate your desired target environment before running this command. If `maturin` is missing, install it with `pip install maturin`.

4. Append the path of `ni-streamer/py_api` to your Python path. The safest way is to do it locally in your script:
    ```Python
    import sys
    import os
    sys.path.append(os.path.join(r'/absolute/path/to/ni-streamer/py_api'))
    ```  
   Alternatively, you can append the path directly to the system-wide `PYTHONPATH` variable.  

5. (Optional) If you want to add custom waveform functions, you can enable the "user function library" feature. First, create your own fork of `usr-fn-lib` repo and git-clone it to your local machine with  
    ```
    git clone https://github.com/your-own-fork-address/usr-fn-lib.git
    ```
    Then compile and install the back-end as in step (3), but with the additional `--features usr_fn_lib` flag:  
     ```
     cargo build --release --features usr_fn_lib
     maturin develop --release --features usr_fn_lib
     ``` 

You should now be able to import and use the streamer:  
```Python
from nistreamer import NIStreamer, std_fn_lib 
from nistreamer import usr_fn_lib  # only if this feature was enabled
```  
See the [demos](https://github.com/pulse-streamer/ni-streamer/tree/main/py_api/demo) for a quick-start guide.

## Project structure

![Project structure schematics. Rust backend is comprised of base-streamer, ni-streamer, and usr-fn-lib crates and implements the streamer-device-channel-instruction tree structure. There is an interface layer between Rust back-end and Python front-end based on PyO3 and maturin. The user-facing Python API part is a collection of proxy classes which artificially "re-inflate" the streamer-device-channel tree which had to be flatten when passing across the language interface.](https://github.com/user-attachments/assets/bf06c51c-393c-47c9-a747-753f97a9f99d)

* `base-streamer` repo:
    * Base traits for channel, device, and streamer types;
    * Base waveform function traits (`Calc<T>`, `FnTraitSet<T>`) and instruction type;
    * Procedural macros for waveform function libraries;
    * Built-in "standard library" of waveform functions.  
    
* `ni-streamer` repo:
    * Concrete implementations for NI DAQmx devices
    * Multi-threading implementation
    * Rust-Python interface layer (flattened Rust API, PyO3 wrapper)
    * User-facing Python API (a set of shallow proxy classes recovering object-oriented channel-device-streamer structure)
    
* `usr-fn-lib` repo (optional dependency) - a template for custom waveform function library. Users are expected to create and maintain their own fork of this repo.

## Main limitations and caveats

* Both Analog and Digital **output** channels are supported. There is no built-in support for **input** channels. However, your application can still read inputs by directly creating NI DAQmx tasks with [PyDAQmx](https://pythonhosted.org/PyDAQmx/) or [nidaqmx](https://nidaqmx-python.readthedocs.io/en/stable/) Python packages as long as they don't conflict with the tasks created by `NIStreamer`.

* Each pulse is specified by start/stop times and the waveform function. Only the waveforms from the built-in library can be generated - somewhat analogous to a [function generator](https://en.wikipedia.org/wiki/Function_generator) as opposed to an [arbitrary waveform generator](https://en.wikipedia.org/wiki/Arbitrary_waveform_generator). But users can add arbitrary custom functions using the optional `usr_fn_lib` feature. It requires writing a minimal amount of Rust code and re-compiling the backend, but we tried to make it maximally simple.

* With the streaming approach, there is always a risk of buffer underrun. Probability can be made very small and not be limiting anymore (something else in the experiment would break more often), but it will always be finite.   
  
  If such a risk is not acceptable, you can effectively disable streaming - specify the buffer size to be equal or greater than the full sequence duration and the entire sequence will be sampled before generation starts. But, compared to the streamed mode, it will take longer to start and may result in a huge amount of RAM to store all the samples.  


* Streaming also means there is no simple way of fast conditional branching (changing the pulse sequence on-the-fly) - the waveform has to be pre-sampled in advance for at least for the whole buffer duration (typically about 100 ms) before writing to hardware. This limitation is mostly relevant for advanced applications like quantum error correction experiments.  
  
  Some limited branching can still be done by hacks. For example, one can use different channels to play the alternative sequences and use a physical switch to select between them. Alternatively, one can configure `NIStreamer` to use an external sample clock signal and gate it to freeze generation at any time for an arbitrary duration. 

