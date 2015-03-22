# LED-
for (i=1:80)
{
 for (j=1:85)
{
z_span = %z span%;
span= %span%;
addcustom;
set("x", span+i*p);
set("y", span+j*p);
set("z",0);
set("name","cone");
set("create 3D object by","revolution");
set("z span",z_span);

set("x span",2*z_span*tan(theta*pi/360));
set("y span",2*z_span*tan(theta*pi/360));
eqn = num2str(tan(theta*pi/360))+"*(x+"+num2str(z_span*(0.5)*1e6)+")";
set("equation 1",eqn); #equation of line to be revolved
#------example of setting rotations-------
set("first axis","y");
set("rotation 1",90);
set("material",material);
if(get("material")=="<Object defined dielectric>") 
{ set("index",index); }




}

 
}




closeall; clear;
run;
runsweep; # remove this line if the sweep has already been run
#save;


# no pattern
np_E2_far = getsweepresult("nopattern_dipole_orientation","E2_far");
np_f_far = getsweepresult("nopattern_dipole_orientation","f_far");
np_dipolePower = getsweepresult("nopattern_dipole_orientation","dipolePower");

# pattern
E2_far = getsweepresult("pattern_dipole_position","E2_far");
f_far = getsweepresult("pattern_dipole_position","f_far");
dipolePower = getsweepresult("pattern_dipole_position","dipolePower");
l     = E2_far.f;
theta = E2_far.theta;



# calculate transmission into 5 deg cone normal to the surface
ext_enhancement_5_deg = matrix(length(l));
res = length(theta);
half_angle = 5;

for (i=1:length(l)){
  E2_far_temp = E2_far.E2_far2;
  E2_far_temp = E2_far_temp(1:res,i);
  np_E2_far_temp = np_E2_far.E2_far2;
  np_E2_far_temp = np_E2_far_temp(1:res,i);
  theta = E2_far.theta;

  ext_enhancement_5_deg(i) = farfield2dintegrate(   E2_far_temp,theta,half_angle,0)/
	                  farfield2dintegrate(np_E2_far_temp,theta,half_angle,0);
}

# for the patterned case, plot fraction of power transmitted 
# into the air, trapped in the glass substrate, and trapped
# in the LED structure

Power_total = dipolePower.dipole_power    / dipolePower.dipole_power;
Power_air   = f_far.f_far2                / dipolePower.dipole_power;
Power_glass = (f_far.f_far1-f_far.f_far2) / dipolePower.dipole_power;
Power_LED  = Power_total - Power_air-Power_glass;



# plot results
plot(c/l*1e6,np_dipolePower.dipole_power,np_dipolePower.dipole_power_box,dipolePower.dipole_power,dipolePower.dipole_power_box,
	"wavelength(um)","dipole power","Internal Quantum Efficiency");
legend("no PC dipole","no PC box","PC dipole","PC box");
plot(c/l*1e6,ext_enhancement_5_deg,"wavelength(um)","f_PC/f_noPC (5 deg cone)","Extraction Efficiency Enhancement");
image(np_E2_far.theta,c/np_E2_far.l*1e6,np_E2_far.E2_far2,"theta (deg)","wavelength (um)","far field in air, no PC");
image(E2_far.theta,c/E2_far.f*1e6,E2_far.E2_far2,"theta (deg)","wavelength (um)","far field in air, PC");
plot(c/l*1e6,Power_total,Power_air, Power_glass,Power_OLED,
	"wavelength(um)","Fraction of emitted power","Patterned LED");
legend("Total","Air","Glass","LED");

