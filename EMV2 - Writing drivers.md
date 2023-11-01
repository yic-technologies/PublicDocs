# EMViewer 2 - Writing drivers

**This guide is only accurate from EMViewer version 12.6.0.** Please make sure you have the latest update!

This page describes how to create a .json driver for use in EMViewer 2. Please follow this guide carefully when creating drivers.

### 1. Prerequisites

Only spectrum analyzers controllable with SCPI commands through a VISA vendor can have drivers added through this method.

Furthermore, the device should be SCPI compliant, as in it can use the *RST and *WAI commands.

**Example:** if you see the spectrum analyzer in NI-MAX, RIGOL UltraSigma or Keysight Connection Expert, then you can write a JSON instruction set for it. Analyzers that require a separate software such as the SignalHound BB60 have to be hardcoded by YIC Technologies.

### 2. Create your JSON file

Create a blank text file and change the extension to `.json`, for example, `MySpectrumAnalyzer.json`. You may name this file whatever you want as long as it does not conflict with another file. We recommend to use `DriverAuthor-AnalyzerFamily.json` (eg. `YICTechnologies-KeysightFieldFox.json`).

Open the file with a text editor such as Notepad, Notepad++, Visual Studio Code, etc.

Copy the following template into the file you have created.

```
{
	"Name" : "DRIVER NAME",
	"Author" : "AUTHOR NAME",
	"Format" : 1,
	"Requires" : {
		"MatchManufacturer" : "MANUFACTURER NAME HERE",
		"MatchName" : "ANALYZER NAME HERE"
	},
	"Commands" : {
		"FrequencyStart" : "FREQ:STAR",
		"FrequencyStop" : "FREQ:STOP",
		"Bandwidth" : "BAND",
		"Single" : "INIT",
		"Continuous" : "INIT:CONT",
		"FormatByteOrder" : "FORM:BORD NORM",
		"FormatBinary" : "FORM REAL,32",
		"SweepPoints" : "SWE:POIN",
		"Fetch" : "TRACE:DATA?"
	}
}
```

In the following sections we will describe the purpose for each block.

### 3. `"Name"`, `"Author"` and `"Version"`

These are used to describe the command set file itself.

Let's assume I'm writing a command set for the RIGOL DSA800 series spectrum analyzer. Put a suitable name, and put your own name (or company) in the author string.

```
	"Name" : "RIGOL DSA800",
	"Author" : "Y.I.C. Technologies",
	"Format" : 1,
```

`Format` tells EMViewer how commands are structured. In future versions, we could require different parameters, so to stay backwards compatible make sure your format version matches the template's.

### 4. `"Requires"`

The requires block is how EMViewer determines which command set should be used for a specific analyzer. It uses **regex** to match the spectrum analyzer's name and manufacturer.

To get an SA's name and manufacturer, it uses an `*IDN?` command. This returns a comma-separated string:

`Manufacturer,Name,Serial Number,Firmware Ver`

For example, the RIGOL DSA875 may return:

```
-> *IDN?
<- Rigol Technologies,DSA875,DSA8E163700013,00.01.06.00.03
```

Only the **manufacturer** and **name** are important for this section.

For the `"MatchManufacturer"` and `"MatchName"` fields, you have two options.

1. **Exact Match**

Write the exact manufacturer and name:
```
    "Requires" : {
		"MatchManufacturer" : "Rigol Technologies",
		"MatchName" : "DSA875"
	},
```

2. **General Match**

or write a regex expression that can identify an entire family of spectrum analyzers. The DSA800 family also contains the DSA832E-TG, the DSA815, etc. So we can write:
```
    "Requires" : {
		"MatchManufacturer" : "Rigol Technologies",
		"MatchName" : "DSA8[0-9]{2}E?(-TG)?"
	},
```

* `DSA8`: matches "DSA8" exactly
* `[0-9]{2}`: matches any two digits
* `E?`: matches "E" (optional)
* `(-TG)?`: matches "-TG" (optional) 

You can view this example [live here](https://regex101.com/r/CAegms/1).

### 5. `"Commands"`

You must input which commands the software should use here. Only a small number of commands are required for spectrum analysis, however the software will make use of some additional commands if they are available.

If you need to use a \" symbol (quotation marks) then make sure to precede it with a \\ (backslash); for example: `INST:SEL "SA"` should be written as `INST:SEL \"SA\"`

Commands are not case-sensitive, and you may use short form or long form SCPI commands, where applicable for your spectrum analyzer. For example, instead of `FREQ:CENT` you may write `FREQuency:CENTer` or `Frequency:Center`.

Below are the commands from our template. These are the minimum required commands.

```
	"Commands" : {
		"FrequencyStart" : "FREQ:STAR",
		"FrequencyStop" : "FREQ:STOP",
		"Bandwidth" : "BAND",
		"Single" : "INIT",
		"Continuous" : "INIT:CONT",
		"FormatByteOrder" : "FORM:BORD NORM",
		"FormatBinary" : "FORM REAL,32",
		"SweepPoints" : "SWE:POIN",
		"Fetch" : "TRACE:DATA?"
	}
```

The following table describes the usage of each command. You must implement the command if the required column is Yes. EMViewer will add parameters to each command when it uses them, so make sure to match the format of the **Example** column.

|Key|Required|Usage|Example|
|-|-|-|-|
|SystemError|Yes|Queries the SA if a command failed to run. Useful for debugging. EMViewer appends `?` to this command.|SYST:ERR|
|ModeSA|No|Changes the device to spectrum analyzer mode. This is important if your device may not be in SA mode when a scan is run.|INST:SEL \"SA\"|
|FrequencyStart|Yes|Sets the start frequency for sweeps.|FREQ:STAR|
|FrequencyStop|Yes|Sets the last frequency for sweeps.|FREQ:STOP|
|Bandwidth|Yes|Sets the RBW.|BAND|
|BandwidthAuto|No|Sets the RBW to be automatic which some SAs can do.|BAND:AUTO|
|Single|Yes|Initialises a single sweep.|INIT|
|Continuous|No|Sets the SA to run sweeps continuously. This is used on scan end.|INIT:CONT|
|PowerUnit|Yes|Sets the unit of power to dBm.|AMPL:UNIT|
|PreampEnable|Yes|Enables or disables the pre-amplifier. EMViewer inserts a 0 or 1 after this command.|POW:GAIN|
|AttenuationAuto|No|Sets the attenuation to be automatic.|POW:ATT:AUTO|
|AttenuationSet|Yes|Sets the attenuation value. EMViewer appends a 0, 10, 20 or 30.|POW:ATT|
|FormatByteOrder|Yes|Tells the spectrum analyzer to output in little endian (LSB). This is **very** important! If the byte order is wrong your results will be extremely out of scale.|FORM:BORD NORM|
|FormatBinary|Yes|Tells the spectrum analyzer to output in binary instead of ASCII text.|FORM REAL,32|
|SweepPoints|Yes|Sets or reads the number of sweep points to use.|SWE:POIN|
|Fetch|Yes|Retrieves the data from the SA.|TRACE:DATA?|
|DisplayEnable|No|If the SA has a screen. Disabling the display often increases the speed of the analyzer.|DISP:ENAB|
|AudioVolume|No|This is used to mute the system if beeps are frequently emitted.|SYST:AUD:VOL|

Any features not seen here will not be changed by EMViewer (unless a `*RST` command is used). If you would like a feature to be added, contact Y.I.C. Technologies.

Finally, save your file to `%appdata%/YIC Technologies/EMViewer2/analyzers/{YourFile}.json`.

If you need support, contact [support@yictechnologies.com](support@yictechnologies.com).