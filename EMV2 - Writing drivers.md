# EMViewer 2 - Writing drivers

This page describes how to create a .json driver for use in EMViewer 2. Please follow this guide carefully when creating drivers.

### 1. Prerequisites

Only spectrum analyzers controllable with SCPI commands through a VISA vendor can have drivers added through this method.

**Example:** if you see the spectrum analyzer in NI-MAX, RIGOL UltraSigma or Keysight Connection Expert, then you can write a JSON driver for it. Analyzers that require a separate software such as the SignalHound BB60 have to be hardcoded by YIC Technologies.

### 2. Create your JSON file

Create a file called `xxx.json`. You may name this file whatever you want as long as it does not conflict with another file. Usually we name them after the device (eg. `DSA875.json`).

Open the file with a text editor such as Notepad.

Copy the following template into the file you have created.

```
{
	"Name" : "DRIVER NAME",
	"Requires" : {
		"MatchManufacturer" : "YIC Technologies",
		"MatchName" : "^SA[0-9]{3}"
	},
	"Commands" : {
		"ModeSA" : "INST:SEL \"SA\"",
		"FrequencySpan" : "FREQ:SPAN",
		"FrequencyCenter" : "FREQ:CENT",
		"FrequencyStart" : "FREQ:STAR",
		"FrequencyStop" : "FREQ:STOP",
		"Bandwidth" : "BAND",
		"BandwidthAuto" : "BAND:AUTO",
		"Single" : "INIT",
		"Continuous" : "INIT:CONT",
		"PowerUnit" : "AMPL:UNIT",
		"PreampEnable" : "POW:GAIN",
		"AttenuationAuto" : "POW:ATT:AUTO",
		"AttenuationSet" : "POW:ATT",
		"FormatByteOrder" : "FORM:BORD NORM",
		"FormatBinary" : "FORM REAL,32",
		"SweepPoints" : "SWE:POIN",
		"Fetch" : "TRACE:DATA?",
		"DisplayEnable" : "DISP:ENAB",
		"AudioVolume" : "SYST:AUD:VOL"
	}
}
```

### 3. Set each attribute

You must now set each attribute to the relevant value for your spectrum analyzer.

```
	"Name" : "DRIVER NAME",
```

Replace the DRIVER NAME with a name of your choice. For example, if our spectrum analyzers started with SA with 3 numbers after that (SA365, SA009, etc), we may name the driver: **YIC Technologies - SA###**.

```
    "Requires" : {
		"MatchManufacturer" : "YIC Technologies",
		"MatchName" : "^SA[0-9]{3}"
	},
```

The `"Requires"` table is how EMViewer determines which driver to use. It will query the spectrum analyzer's manufacturer and name using an `*IDN?` command, and compare those to the driver. For example, if our spectrum analyzer returned this:

```

```