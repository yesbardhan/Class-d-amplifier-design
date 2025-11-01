* Class-D amplifier demo netlist for ngspice
* Notes: This is a functional, simplified simulation for validation of PWM approach.
* It uses idealized MOSFET models for clarity. Replace with vendor models for more realistic switching loss estimates.


* ---------------------------------------------
* Power rails and load
Vp vdd 0 DC 24
Vm vss 0 DC -24
Rload out 0 8
* ---------------------------------------------
* Audio input: 1 kHz sine, 1 Vrms (2*sqrt(2) sets)
Vin in 0 SIN(0 1 1k)


* ---------------------------------------------
* Triangle generator: opamp integrator + comparator (ideal opamps using e-source behavioural)
* Simple relaxation oscillator: integrator integrates square to get triangle
R1 in1 0 1meg
* We'll emulate a comparator with a behavioural source
* Triangle carrier (vc) approx using a voltage-controlled source oscillator
Etri tri 0 value={V(tri)}
* For simplicity in ngspice, below we implement a behavioural triangular source:
Btri tri 0 v=abs(1-2*fract( time*250000 ))
* This B-source creates a triangle between 0 and 1 at 250 kHz (carrier)
* Scale and offset below
Vtri_dc vtri 0 DC 0
Etri1 vc 0 value={2*(V(tri)-0.5)}
* Now vc is approx -1..+1 at 250kHz


* ---------------------------------------------
* Modulator comparator: compare audio (scaled) to triangle carrier
* Scale Vin so audio ±4V compared to carrier ±1V -> preamp gain
Gpre inp in 0 in 0 1
* Implement audio gain with a behavioural source
Bpre inp 0 v=V(in)*3


* Comparator output (ideal): pwm node pwm is +Vdd when audio>carrier else -Vdd
Bcmp pwm 0 v=V(+vdd)*(V(inp)-V(vc) > 0 ? 1 : -1)


* But SPICE conditional inside B-source differs: use a B-source expression using if( )
Bcmp pwm 0 v=if(V(inp)-V(vc) > 0, V(vdd), V(vss))


* ---------------------------------------------
* Output stage: two MOSFETs as push-pull controlled by pwm
* Use simple NMOS and PMOS? We'll use two N-channel MOSFETs: top driven to vdd or float.
* For this simplified sim, implement switches controlled by pwm using voltage-controlled switch
S1 vdd out pwm vdd swmod
S2 out 0 pwm 0 swmod
.model swmod SW(Ron=0.05 Roff=1Meg Vt=0.5)


* Add small LC low-pass filter (LC cutoff ~50 kHz) to recover audio
L1 out_f out 20u
C1 out 0 20n
Rseries out out_f 0.01
* connect filtered node to load
Rload out 0 8


.control
tran 200u 2m
plot V(out)
.endc


.end
