> [!WARNING]
> This project is a work-in-progress - already usable, but some features are still missing and breaking changes are still possible. Documentation is coming soon.

## NI Pulse Streamer

* High-level Python API for pulse sequence generation with [National Instruments](http://www.ni.com/) DAQ hardware.

* Package format - can be imported and used in any Python code ranging from a Jupyter Notebook script to a GUI app.

* Rust backend handles multithreading and on-the-fly sample calculation and streaming - only a small portion of the sequence is sampled at a time. As a result, total sequence duration is practically unlimited and all cards can simply run at the maximal sampling rate.

* See [demos](https://github.com/pulse-streamer/ni-streamer/tree/main/py_api/demo) for the example workflow (these demos are outdated but still give a good idea)

### Main limitations and caveats to be aware of:

* Both Analog and Digital _output_ channels are supported. There is no built-in support for _input_ channels. However, your application can still read inputs by directly creating NI DAQmx tasks with [PyDAQmx](https://pythonhosted.org/PyDAQmx/) or [nidaqmx](https://nidaqmx-python.readthedocs.io/en/stable/) Python packages as long as they don't conflict with the tasks created by `NIStreamer`.

* Each pulse is specified by start/stop times and the waveform function. Only the waveforms from the built-in library can be generated - somewhat analogous to a [function generator](https://en.wikipedia.org/wiki/Function_generator) as opposed to an [arbitrary waveform generator](https://en.wikipedia.org/wiki/Arbitrary_waveform_generator). But users can add arbitrary custom functions using the optional `usr_fn_lib` feature. It requires writing a minimal amount of Rust code and re-compiling the backend, but we are trying to make it maximally simple.

* Since `NIStreamer` relies on streaming, there is no simple way of doing fast conditional branching (changing the pulse sequence on-the-fly) - the waveform has to be pre-sampled in advance at least for the whole buffer duration (typically about 100 ms) before writing to hardware. This limitation is mostly relevant for advanced applications like quantum error correction experiments.  
Some limited branching can still be done by hacks. For example, one can use different channels to play the alternative sequences and use a physical switch to select between them. Alternatively, one can configure `NIStreamer` to use an external sample clock signal and gate it to freeze generation at any time for an arbitrary duration. 
 
<!--

**Here are some ideas to get you started:**

🙋‍♀️ A short introduction - what is your organization all about?
🌈 Contribution guidelines - how can the community get involved?
👩‍💻 Useful resources - where can the community find your docs? Is there anything else the community should know?
🍿 Fun facts - what does your team eat for breakfast?
🧙 Remember, you can do mighty things with the power of [Markdown](https://docs.github.com/github/writing-on-github/getting-started-with-writing-and-formatting-on-github/basic-writing-and-formatting-syntax)
-->
