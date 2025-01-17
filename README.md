# PWL files simulate contact bounce in LTspice

[LTspice](https://www.analog.com/en/design-center/design-tools-and-calculators/ltspice-simulator.html) is a free (not open, but free) circuit analysis program maintained by Linear Technology, and their *corporate parent* Analog Devices. A [PWL ("Piece-Wise Linear") file](https://www.analog.com/en/technical-articles/ltspice-piecewise-linear-functions-for-voltage-current-sources.html) describes a *custom waveform* that may serve as an input source (voltage or current) in LTspice, and circuit performance to that input may be analyzed. Looked at this way, a PWL file may be useful in predicting the response of a circuit to non-typical or random inputs. For example, how does a circuit react to ***"contact bounce"*** when a switch is closed (or opened) while connected to that circuit?    

The LTspice documentation provides information on methods for creating a PWL file [here](https://www.analog.com/en/technical-articles/ltspice-importing-exporting-pwl-data.html) - and [here](https://www.analog.com/en/technical-articles/ltspice-piecewise-linear-functions-for-voltage-current-sources.html). These methods work fine for some waveforms, but may be very tedious for creating a high fidelity model of contact bounce. Further, it has been shown that [each switch has its own unique way of bouncing](https://www.eejournal.com/article/ultimate-guide-to-switch-debounce-part-1/), and so there is no such thing as a "general" - or *"one-size-fits-all"* model for contact bounce. All of these things (and more) were on my mind when I resolved to attempt to create a [*better mousetrap*](https://idioms.thefreedictionary.com/build+a+better+mousetrap). 

Conceptually, my mousetrap was simple, and not even innovative: Capture contact bounce on an oscilloscope, and ***export*** the waveform data to PWL data format. I had a few manual toggle switches on hand, and so I set up [my scope](https://www.keysight.com/en/pdx-x201760-pn-MSO9104A/mixed-signal-oscilloscope-1-ghz-4-analog-plus-16-digital-channels?nid=-32534.1150138&cc=US&lc=eng&pm=ov) to capture the bounce. It wasn't difficult to set up the measurement (here's a file exported from the scope detailing my setup), and soon I was capturing all sorts of [*random* waveforms](https://imgur.com/a/JcRDZ7k). It was fascinating to see how much variance existed in each closure of the same switch! 

| 1. Scope Measurement Screenshot | 2. Scope Measurement Screenshot & Comments |
| -------------------------- | -------------------------- |
<img src="pix/ContactBounce07.png" alt="Scope Measurement Screenshot, Contact Bounce Measurement 7" width="480">|<img src="pix/ContactBounce07-Comments.png" alt="Scope Measurement Screenshot w/Cmts, CB07" width="480">

Once I exported the first data file from the oscilloscope, I realized constructing the PWL file would be more challenging than it seemed at first. From the scope's *export menu* I chose the CSV format. A short time later I had a 277MB file with 10 million data points (voltage vs time) at 0.2 nanosecond intervals. This was far more data than needed, and far more data than LTspice could handle on my Mac<sup id="a1">[Note 1](#f1)</sup>. Here's the [rather large CSV file](https://drive.google.com/file/d/14TgyNHGOWcfiwsI2c3uICQXNFt6SBPWj/view?usp=sharing) containing all the data points for the contact bounce measurement #7 pictured in the screenshot above (CB07).  

And so, here's the challenge: ***How to create a small-ish PWL file for LTspice from a 277MB CSV file downloaded from the oscilloscope?*** 

 [`awk`](https://en.wikipedia.org/wiki/AWK) seemed a good choice, but after struggling a couple of days, I [called for help](https://unix.stackexchange.com/questions/627800/can-awk-sum-a-column-over-a-specified-number-of-lines) - promptly answered with a  functional, well-written block of code. I've hacked it a bit based on things learned since then, and likely to continue - the latest is listed in the repo above. 

The format for a PWL file is illustrated below:

```
Col. 1   Col. 2
time     voltage 
0.0      0.0
2.0E-06  0.0
2.01E-06 1.9
2.02E-06 3.3
8.0E-06  3.3
9.0E-06  0.0
20.E-06  0.0
22.E-06  1.8
24.E-06  3.3
36.E-06  3.3
37.E-06  0.0
50.E-06  0.0
```

LTspice can read values in scientific notation or floating point, and one or more spaces serve to separate the time value from the voltage (or current) value. LTspice will interpret the data points as a *waveform*, and can display them in the *Waveform Viewer*: 







---

## Notes: 

   * <b id="f1">Note 1 : </b>In response to a query re size limits on PWL files, an Analog Devices support engineer has suggested that 500K data points is a practical limit. [↩](#a1) 
   
## Description of transformation signal proces from oscilloscope References #9 [Added by mamin27]:

* Source file is generated by oscilloscope application, References #10:
  - Import the output file into LibreOffice Calc application as scv file
  - modify (add and tailor new tabs as it is in a example) and save transformation file to oscillo_src.ods
* The LibreOffice Calc application was used to generate final `csv file` from 2 tab:
  - add and tailor additional tabs csv_data_CH1 or csv_data_CH 
  - save final csv output file oscillo_src.csv from above tabs
  - use vim or similar tool to modify final CSV file because awk script use different notation:
    -  replace ',' to '.' character `[:1,$s/,/./g]` 
    -  replace ';' to ',' character `[:1,$s/;/,/g]`
  - here is final output format oscillo_src.csv used by `awk script` 

```ps
comet@raspberrypi:~/LTspice-PWLfiles $ cat oscillo_src.csv
5.00E-06,11.400E+00
1.00E-05,11.400E+00
1.50E-05,11.400E+00
2.00E-05,11.400E+00
2.50E-05,11.400E+00
```

thin-pw1.awk (new) `AWK script` generate pwl file

* Description of `AWK scripts`:

  - The thin-pwl0.awk script use a `data rounding logic` and it is developed by origin author seamusdemora - Reference #11 .
  - The thin-pwl1.awk script use a `duplicates elimination logic` for transformation signal.

Note:
In example directory you find files that you use for transformation proces from oscilloscope source file to final LTspice PWL file.

commands:

```ps
comet@raspberrypi:~/LTspice-PWLfiles $ awk -f ./thin-pwl0.awk ./example/oscillo_src.csv > ./example/oscillo_0.pwl
comet@raspberrypi:~/LTspice-PWLfiles $ awk -f ./thin-pwl1.awk ./example/oscillo_src.csv > ./example/oscillo_1.pwl
```

![compare_conversion_strategy](https://user-images.githubusercontent.com/26118162/213135139-fff3cfd5-2058-4b41-9bdb-b3db7b420ec5.PNG)

Green line (logic thin-pwl0.awk), blue line (logic thin-pwl1.awk)

* Use PWL file in LTSpice application
![LTSpice](https://user-images.githubusercontent.com/26118162/213154897-1df16673-ebb4-4f65-8f61-645b3de88a49.jpg)

## References, Resources & Further Reading

1. [Max Maxfield's 9-part saga on debouncing, which includes both hardware **and** software solutions](https://www.eejournal.com/article/ultimate-guide-to-switch-debounce-part-5/) 
2. [Jack Ganssle's 2-part article, which also offers a hardware & software solutions](http://www.ganssle.com/debouncing-pt2.htm)  
3. [Recognizing and Coping with Contact Bounce on the Raspberry Pi](https://www.dummies.com/computers/raspberry-pi/recognizing-and-coping-with-contact-bounce-on-the-raspberry-pi/) 
4. [Switch Bounce and How to Deal with It](https://www.allaboutcircuits.com/technical-articles/switch-bounce-how-to-deal-with-it/) 
5. [What is Switch Bouncing and How to prevent it using Debounce Circuit](https://circuitdigest.com/electronic-circuits/what-is-switch-bouncing-and-how-to-prevent-it-using-debounce-circuit) 
6. [LTspice and mechanical switch problem](https://www.eevblog.com/forum/beginners/ltspice-and-mechanical-switch-problem/msg3162872/#msg3162872) 
7. [Anyone have a .pwl file describing contact bounce for use in LTspice?](https://www.eevblog.com/forum/projects/anyone-have-a-pwl-file-describing-contact-bounce-for-use-in-ltspice/msg3370850/#msg3370850) 
8. [Q&A: What is the proper way to debounce a GPIO input?](https://raspberrypi.stackexchange.com/questions/118349/what-is-the-proper-way-to-debounce-a-gpio-input) 
9. [GDS-1202B - Osciloskop, 2x 200MHz, 1GSPS, GW Instek](https://www.gwinstek.com/en-global/products/detail/GDS-1000B)
10. [OpenWave project](https://github.com/mamin27/OpenWave-1KB)
11. [Origin LTspice-PWLfiles project](https://github.com/seamusdemora/LTspice-PWLfiles)
