# Introduction
BTT SB22xx boards suffer from a recurring problem, crashing the print:
- SB2209: `"TMC 'extruder' reports error: GSTAT: 00000001 reset=1(Reset)`
- SB2240: `"TMC 'extruder' reports error: GSTAT: 00000004 uv_cp=1 (undervoltage!)`

It's quite possible to print for hundreds of hours without any problems, and then, at random, have this failure several times in a row. A few hours later, the problem may have disappeared without explanation.

Anyone who builds a Voron knows that it's not going to be all fun and games. But this breakdown is particularly frustrating, because it seems unsolvable. But the community around [ticket 28 on GitHub](https://github.com/bigtreetech/EBB/issues/28) has been quite prolific. And even if BTT proves to be totally incompetent and lax, shame on them, I remain convinced that we can find the solution ourselves.

The purpose of this page is to leave a report on the solution I'm proposing, as well as the process that led me to it. As it stands, I haven't had a single error since I connected the extruder motor case to mains earth. But that doesn't mean the problem has been completely solved.

Don't expect miracles either, if you've neglected the wiring or a good quality power supply with sufficient power.

Comments are welcome, but only constructive remarks. At this stage, the solution I'm proposing is perfectly functional, at least for me, until further notice. I've already wasted over a hundred hours on this problem, and spent a whole day writing this document for the benefit of the community, and I'm not going to waste any more on assumptions or unfounded beliefs. If you want to comment, do so with a rigorous description of your observations and relevant questions or assertions, as well as the reasoning that led you to them.

# Rumors and beliefs

## BTT lack any support
**$\color{green}{\textsf{True}}$** - I did get very few answers from BTT, after sending an exhaustive report, for which they complimented me. But since then they've given no sign of life, ignoring all requests to join our efforts. Now I've decided to invest my time in the community, and they can go to hell (actually, at the moment, we're the ones who are).

## SB22xx is a piece of c...
**$\color{red}{\textsf{Wrong}}$** (my opinion) - As far I could compare, all suppliers use quite the same design, as described in [Trinamic's manual](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc2240_datasheet.pdf). And the board is quite well made.

## BTT did release a newer version of the board that could fix the problem
**$\color{red}{\textsf{Wrong}}$** - I did ask BTT about a newer version, and here is the answer (Feb. 4th, 2024):

>Currently we are still trying to find the reason why the undervoltage situation keeps on happening, we had tested a lot of times with a lot of devices, but it just didn't occur. It could be a suitabiluty issue? We will keep on reserching it.

Although, some people pretend they did release a version 1.1, but no one seems to know what they did change. But it's not related to this issue, for sure.

## Replacing the SB2240 by an SB2209 could solve the issue
**$\color{red}{\textsf{Wrong}}$** - I did spend tens of hours on this issue, switching several times between SB2240 and SB2209. The SB2240 shows an "undervoltage" error.  The SB2209 has much less detection capabilities that SB2240, and only shows "RESET". But both of them crash in the exact same behavior.

## Using the RP2040 version might help
**$\color{red}{\textsf{Wrong}}$** - Seriously ? Do you really believe that replacing the MCU will change anything in the driver behavior ?

## CAN BUS is unstable and should not be used
**$\color{red}{\textsf{Wrong}}$** - It's very easy to know if CAN BUS is involved, and on my behalf, all error counters keep showing 0:
```sh
ifconfig can0
can0: flags=193<UP,RUNNING,NOARP>  mtu 16
        unspec 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  txqueuelen 1024  (UNSPEC)
        RX packets 860892  bytes 5417469 (5.1 MiB)
        RX errors 0  dropped 266  overruns 0  frame 0
        TX packets 1590609  bytes 11310936 (10.7 MiB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

## Problem can be fixed by changing the driver settings
**$\color{orange}{\textsf{Doubt}}$** - Some people did give it a try (ex. [this quite exhausting example](https://github.com/Esoterical/voron_canbus/blob/main/toolhead_flashing/common_hardware/BigTreeTech%20SB2209%20and%20SB2240/SB2240%20Stepper%20Configuration%20and%20Undervoltage.md)).

I did try on my own, whithout any success. I don't have the knowledge to know what the impact will be, so I'd rather not touch what I don't understand. Furthermore, those settings only apply on SB2240, and won't solve the SB2209 RESET issue. Using a workaround is not a reliable solution, without fixing the cause first.

## Using a driver on the mainboard is the solution
**$\color{green}{\textsf{True}}$** - This solution seems fairly obvious, since before installing SB22xx, there were no errors.

However, that doesn't seem quite as true as it sounds. I've read in some forums (need links) that the problem has also been seen, rarely, on mainboards. But there could be many other causes, such as bad wiring.

As the final clue is an ESD issue, this problem should be solved first. Even minimized by the cables length, ESD is destructive for electronics.

# Hypothesis and testings

## Bad cables / connectors
First thing first, cables and connectors are the main cause of so many issues. I did change the CAN BUS cable, rebuilt and triple checked all connectors, but the problem persists.

## Power supply
My power supply was a MeanWell LRS-200-24. After increasing the accelerations and replacing the heating element to 80W, I was getting close to the limit of the power supply. I did replace it by a LRS-350-24, but the problem perists.

## Defective extruder motor
When in doubt, I did replace the motor, but no change.

##  Driver is overheating
This idea mainly concerns the SB2209 (error: RESET)

I watched the board for several hours with a thermal camera, without noticing the slightest variation in temperature. As the driver is located on the rear panel, this observation lacks rigor. But there's nothing to suggest that overheating is the cause.

## ESD (ElectroStatic Discharge)
Many people did suggest ESD. And I'm firmly convinced that this is the real cause of the problem.

The reason I'm so confident is that the problem occurred when I was manually pushing the filament into the reverse bowden, when no printing was in progress. At the exact moment when the filament came into contact with the gear, the "undervoltage" error was triggered. And it happened several times.

The ESD hypothesis is the best candidate as the real cause, and explains why this issue is so random and hard to reproduce. Furthermore, ESD is described as a cause that matches quite well our issue in the [Trinamic's manual](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc2240_datasheet.pdf), but not specific to the undervoltage error.

> ESD events occurring during operation can cause a reset or even a destruction of the motor driver

> Especially, plastic housings and belt drive systems tend to cause ESD events of several kV. It is best practice to avoid ESD events by attaching all conductive parts, especially the motors themselves to PCB ground, or to apply electrically conductive plastic parts. 

Keywords:
- ESD events can cause a reset (could apply to SB2209)
- plastic housings and belt drive (Steathburner is often made of ABS)
- attaching all conductive parts, especially the motors themselves to PCB ground

### The belts
Trinamic reminds that belts are cause of static charges.

Although, it was quite a surprise when I did measure their resistance on my Troodon 2.0 (Voron 2.4):
- ohm meter: ~ 6 MΩ
- megohm meter (250 ~ 1000V): < 1 MΩ

Since the chassis is grounded to mains earth, belts don't seem to be the cause of the problem. Pulleys, bearings and screws are conductive enough to dissipate the charge during printing, even if pulleys are part of the charge. This was confirmed by measuring the static charge of the belts while printing, with < 200V.

### Between the hotend and the bed
The heater of the bed is usually directly powered by the mains, and the bed must absolutely be connected the the mains earth. On the hotend side, plastic usually has a lower resistance when melted. In such situation, the hotend has a quite good conductive path to the bed and ground.

I could not measure any charge above 200V on the head while printing, which is a normal value in the air.

## EMI (ElectroMagnetic Interference)
On March 15th, 2024, [someone](https://github.com/bigtreetech/EBB/issues/28#issuecomment-1999028361) did suspect a "common mode interference" to distrurb the driver.

Common mode interference results in small spikes at the start of each pulse of a switching power supply. It generates radio frequencies (EMI), and contributes to ripple (variation in output voltage).

As far I could measure, there is a ripple of 0.7V on the SB22xx, depending on the heater and the motor consumption. This variation has been tolerated without the slightest problem over hundreds of hours of printing.

I have neither the knowledge nor the equipment to measure EMI. But I find this hypothesis unconvincing, in the sense that we should also have had this problem before installing an SB22xx.

## Heatsink
On March 15th, 2024, [someone](https://github.com/bigtreetech/EBB/issues/28#issuecomment-1999520880) did solve the issue by removing the heatsink (placed on outer side of the motor).

I am surprised by this solution, since I cannot understand how this heatsink could interfere.

Observations:
- the heatsing if glued on the top of copper tablet, which is connected to the ground
- the heatsink is at a reasonable distance from connector pins, and does not suggest that it can interfere with or touch the welds
- I could not measure any resistance on the glue of the heatsink (> 1nS, > 10^9 Ohms)

Let's wait if other people report some experience, of if someone has an idee what could be the relation.

# On the track of ESD

## Hyphothesis of the cause
The filament friction into the reverse bowden creates a high static charge of the reverse bowden and the filament itself. I could measure static charges up to 10kV on the reverse bowden, as the filament unwound. Furthermore, Voron and belts, it's a love story. Faster is better, isn't it ?

Please note that the TMC2240 is ESD-protected, but not for strong or long spikes.

> The TMC2240 motor phase output pins are protected up to 8kV HBM in the application when using a bypass capacitor of at least 1uF on the positive voltage supply (VS pins).

When two bodies get in contact, the static chage with balance between both (e.g. a spark from a car handle). But the balancing speed depends on the resistance of the material (e.g. car handles don't trigger spark anymore since they are painted). Plastics have a resistance of more than 10^15 Ohms, considered as insulating. When the plastic gets in contact with metal, which is higly conductive, the energy will travel to the shortest path, the motor cage in this case (which is in contact with the gears).

The motor cage will act as a capacitor, storing evermore energy. At the critical point, the cage will discharge to the shortest path it could find, meaning the motor windings, or maybe the driver itself, which is quite close to the cage.

## The randomness of the cause
Static charges dissipate naturally in the air, depending on its humidity. But when a dryer is used, the air loses its ability to dissipate the charge, increasing the risk of an overload.

In the end, the problem is not as random as it seems. It depends on the type of filament, filament speed, stiffness and angles (cause of frictions), and the moisture in the air and filament.

## Electric resistance of plastics
In terms of ESD, materials are classified according to their electrical resistance as follows.

```
+----------------+------------+-------------+------------+
| Ohms*cm        |    10^1    |    10^6     |  > 10^12   |
+----------------+------------+-------------+------------+
| classification | conductive | dissipative | insulative |
+----------------+------------+-------------+------------+
|                |        antistatic        |            |
+----------------+--------------------------+------------+
```

Most plastics that we print have a resistance > 10^15 Ohms*cm, highly insulative.

Filaments that are ESD specific, or some with CF (Carbon Fiber) are in the range of "antistatic". Some may even be low enough to be conductive. 

## Solution
Assuming that the power supply and wiring are not at fault, the solution is to **pull a dedicated cable from the extruder motor cage to earth**.  [Trinamic's manual](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc2240_datasheet.pdf) recommends a wire between the cage and the PCB ground (minus of the power supply). But I'm not confident with this solution (see next topic, regarding ground).

Since I did pull a wire from the extruder motor cage to mains earth, I've been able to print several hundreds of hours without the slightest problem, a huge improvement. [Several people have reported similar results in the ticket on GitHub](https://github.com/bigtreetech/EBB/issues/28).

But let's not claim victory too soon. If this is not enough, there are still other ideas to explore around ESD:
- printing some back parts of the StealthBurner with ESD or CF filament (not good as grounding the motor with a wire, but could help dissipating into the gantry)
- shielding the driver
- installing capacitors on the driver output, in order to absorb higher spikes, as described in the [Figure 36 of Trinamic's manual](https://www.analog.com/media/en/technical-documentation/data-sheets/tmc2240_datasheet.pdf).

In the event of deadlock, there are still these solutions:
- use a driver from the motherboard
- tune the sensitivity of the TMC (only applies to TMC2240)

## Misunderstanding of the term "ground"
The term of "ground" is a subject of misunderstanding, because it is used in three different meanings:
- the mains earth - the real ground
- the chassis
- the "minus" (0V) of the DC power supply

First of all, the chassis must be connected to the mains earth, as well as the bed (with a dedicated cable). The electrical resistance between segments of the frame is not null, usually few ohms. It is not perfect for security, but good enough for dissipating static charges. Allthough, the conductivity in the frame is not enough for absorbing spikes from the motor cage, which also requires a dedicated cable (trust me, I've made the test).

The DC minus (0V) of the power supply is usually not connected to earth. In the event of an ESD, the mass available to absorb the discharge is limited to the power supply, cables, and electronic circuits, and can cause severe damage. Since I'm not an electronics engineer, I asked a professional for advice if we can connect the DC minus to earth, and he had no objections. Although, I did ask Chat GPT:

> May be connect the DC 0V to mains earth, regarding EU regulation ?

> Connecting the DC 0V (negative terminal) of a switching power supply to mains earth is generally not permitted under EU regulations and is not considered a standard practice in electrical installations. In the EU, electrical safety regulations are designed to minimize the risk of electric shock, fire, and other hazards.
>
> The European Union's Low Voltage Directive (Directive 2014/35/EU) and other related standards emphasize safe electrical design and installation practices. Typically, connecting the negative terminal of a DC power supply to mains earth can introduce safety risks and is likely to be non-compliant with these regulations.
>
> Furthermore, connecting the negative terminal of a DC power supply to mains earth can create potential safety hazards such as short circuits, electrical shocks, and interference with other electrical devices.
>
> It's essential to adhere to EU regulations and standards when designing and installing electrical systems and equipment. If there are specific circumstances where such a connection is necessary, it's crucial to consult with electrical safety authorities or professionals to ensure compliance and mitigate any associated risks.

There is clearly a risk, depending on how the area has been earthed. From my point of view, I prefer to pull an independent wire to ground and take no risk. And as long as the problem seems to have stabilized, I don't see the point of doing so.

# References
- https://github.com/bigtreetech/EBB/issues/28 (SB2240 undervoltage)
- https://github.com/bigtreetech/EBB/issues/47 (SB2209 reset)
- https://klipper.discourse.group/t/tmc-extruder-reports-error-gstat-00000001-reset-1-reset/1156/25
- https://www.professionalplastics.com/professionalplastics/ElectricalPropertiesofPlastics.pdf
