##This contains steps for creating android image for beaglebone.
###We will be using linux kernel 3.8 beacause it has device tree support which can be used for PWM module or ADC without any need to recompile the kernel.
####For this project we are using Robert Nelson's kernel 3.8 and rowboat project to compile the android.


1. Get rowboat's android project for beaglebone
	* mkdir RobertNelsonKernel
	* cd RobertNelsonKernel
	* repo init ­u git://gitorious.org/rowboat/manifest.git ­m rowboat­jb­am335x.xml 
	* repo sync 