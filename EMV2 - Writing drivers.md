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

### 3. Requires block

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

The `"Requires"` table is how EMViewer determines which driver to use. It will query the spectrum analyzer's manufacturer and name using an `*IDN?` command, and compare those to the driver.

For example, let's say we have a spectrum analyzer, use the `*IDN?` command and receive:

```
YIC Technologies,SA193,8873197649,v0.0
```

The software would interpret this as:
* **Manufacturer:** YIC Technologies
* **Name:** SA193

The `"MatchManufacturer"` and `"MatchName"` attributes must contain patterns that match the analyzer we want to use. We may use the manufacturer and name exactly:

```
    "Requires" : {
		"MatchManufacturer" : "YIC Technologies",
		"MatchName" : "SA193"
	},
```
> When only one spectrum analyzer exists using this driver

Or more general:

```
    "Requires" : {
		"MatchManufacturer" : "YIC Technologies",
		"MatchName" : "^SA[0-9]{3}"
	},
```

[Click here](https://regex101.com/r/7VRd2J/1) to see the example online.

### 4. Commands block

You must input which commands the software should use here. Only a small number of commands are required, however the software will make use of some additional commands if they are available.

If you need to use a \" (quotation marks) then make sure to precede it with a \\ (backslash); for example: `INST:SEL "SA"` should be written as `INST:SEL \"SA\"`

Commands are not case-sensitive, and you may use the longer form instead if you wish, where applicable for your spectrum analyzer. For example, instead of `FREQ:CENT` you may write `FREQuency:CENTer` or `Frequency:Center`.

Here is the template:

```
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
```

The following table describes the usage of each command.

|Key|Required|Example|
|-|-|-|
|ModeSA|If the analyzer can be in another mode|INST:SEL \"SA\"|
|FrequencySpan|Yes|FREQ:SPAN|
|FrequencyCenter|Yes|FREQ:CENT|
|FrequencyStart|As an alternative to FrequencySpan|FREQ:STAR|
|FrequencyStop|As an alternative to FrequencySpan|FREQ:STOP|
|Bandwidth|Yes|BAND|
|BandwidthAuto|No|BAND:AUTO|
|Single|Yes|INIT|
|Continuous|No|INIT:CONT|
|PowerUnit|Yes|AMPL:UNIT|
|PreampEnable|Yes|POW:GAIN|
|AttenuationAuto|No|POW:ATT:AUTO|
|AttenuationSet|Yes|POW:ATT|
|FormatByteOrder|Yes|FORM:BORD NORM|
|FormatBinary|Yes|FORM REAL,32|
|SweepPoints|Yes|SWE:POIN|
|Fetch|Yes|TRACE:DATA?|
|DisplayEnable|If the SA has a screen. Disabling the display often increases the speed of the analyzer|DISP:ENAB|
|AudioVolume|No|SYST:AUD:VOL|

The software will use the commands in the driver to control the analyzer. If there is a missing feature (for example: coupling control) please contact YIC Technologies so we can add the feature.