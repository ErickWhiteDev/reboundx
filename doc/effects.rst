.. _effects:

Astrophysical Effects
=====================

Below are descriptions for each of the effects included in REBOUNDx.
Different implementations for the same effect are grouped together.
All effects follow the same recipes for usage, see the Python quick-start guide (:ref:`python_qs`) or C quick-start guide (:ref:`c_qs`).
Probably the quickest way to get up and running is to edit one of the linked examples for the effect you're interested in.

Orbit Modifications
^^^^^^^^^^^^^^^^^^^
REBOUNDx offers two ways of modifying orbital elements (semimajor axis/eccentricity/inclination damping, precession, etc.)
In both cases, each particle is assigned evolution timescales for each orbital element.  
Positive timescales correspond to growth / progression, negative timescales correspond to damping / regression.  
Semimajor axes, eccentricities and inclinations grow / damp exponentially.  
Pericenters and nodes progress/regress linearly.

.. _modify_orbits_forces:

modify_orbits_forces
********************

======================= ===============================================
Authors                 D. Tamayo, H. Rein
Implementation Paper    `Kostov et al., 2016 <https://ui.adsabs.harvard.edu/abs/2016ApJ...832..183K/abstract>`_.
Based on                `Papaloizou & Larwood 2000 <http://labs.adsabs.harvard.edu/adsabs/abs/2000MNRAS.315..823P/>`_.
C Example               :ref:`c_example_modify_orbits`
Python Example          `Migration.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/Migration.ipynb>`_
                        `EccAndIncDamping.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/EccAndIncDamping.ipynb>`_.
======================= ===============================================

This applies physical forces that orbit-average to give exponential growth/decay of the semimajor axis, eccentricity and inclination.
The eccentricity damping keeps the angular momentum constant (corresponding to `p=1` in modify_orbits_direct), which means that eccentricity damping will induce some semimajor axis evolution.
Additionally, eccentricity/inclination damping will induce pericenter/nodal precession.
Both these effects are physical, and the method is more robust for strongly perturbed systems.

**Effect Parameters**

If coordinates not, defaults to using Jacobi coordinates.

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
coordinates (enum)           No          Type of elements to use for modification (Jacobi, barycentric or particle).
                                         See the examples for usage.
============================ =========== ==================================================================

**Particle Parameters**

One can pick and choose which particles have which parameters set.  
For each particle, any unset parameter is ignored.

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
tau_a (double)               No          Semimajor axis exponential growth/damping timescale
tau_e (double)               No          Eccentricity exponential growth/damping timescale
tau_inc (double)             No          Inclination axis exponential growth/damping timescale
============================ =========== ==================================================================


.. _inner_disk_edge:

inner_disk_edge
***************

======================= ================================================================================================================
Authors                 Kajtazi, Kaltrina and D. Petit, C. Antoine
Implementation Paper    `Kajtazi et al 2022 <https://ui.adsabs.harvard.edu/abs/2022arXiv221106181K/abstract>`_.
Based on                `Pichierri et al 2018 <https://ui.adsabs.harvard.edu/abs/2018CeMDA.130...54P/abstract>`_.
C example               :ref:`c_example_inner_disk_edge`
Python example          `InnerDiskEdge.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/InnerDiskEdge.ipynb>`_.
======================= ================================================================================================================

This applies an inner disk edge that functions as a planet trap. Within its width the planet's migration is reversed by an opposite and roughly equal magnitude torque. Thus, stopping further migration and trapping the planet within the width of the trap. 
The functions here provide a way to modify the tau_a timescale in modify_orbits_forces, modify_orbit_direct, and type_I_migration.
Note that the present prescription is very useful for simple simulations when an inner trap is needed during the migration but it shouldn't be considered as a realistic model of the inner edge of a disk.

**Effect Parameters**

============================ =========== ===================================================================================
Field (C type)               Required    Description
============================ =========== ===================================================================================
ide_position (double)        Yes         The position of the inner disk edge in code units 
ide_width (double)           Yes         The disk edge width (planet will stop within ide_width of ide_position)
============================ =========== ===================================================================================


.. _modify_orbits_direct:

modify_orbits_direct
********************

======================= ===============================================
Authors                 D. Tamayo
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_. 
Based on                `Lee & Peale 2002 <http://labs.adsabs.harvard.edu/adsabs/abs/2002ApJ...567..596L/>`_. 
C Example               :ref:`c_example_modify_orbits`
Python Example          `Migration.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/Migration.ipynb>`_,
                        `EccAndIncDamping.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/EccAndIncDamping.ipynb>`_.
======================= ===============================================

This updates particles' positions and velocities between timesteps to achieve the desired changes to the osculating orbital elements (exponential growth/decay for a, e, inc, linear progression/regression for Omega/omega.
This nicely isolates changes to particular osculating elements, making it easier to interpret the resulting dynamics.  
One can also adjust the coupling parameter `p` between eccentricity and semimajor axis evolution, as well as whether the damping is done on Jacobi, barycentric or heliocentric elements.
Since this method changes osculating (i.e., two-body) elements, it can give unphysical results in highly perturbed systems.

**Effect Parameters**

If p is not set, it defaults to 0.  If coordinates not set, defaults to using Jacobi coordinates.

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
p (double)                   No          Coupling parameter between eccentricity and semimajor axis evolution
                                         (see Deck & Batygin 2015). `p=0` corresponds to no coupling, `p=1` to
                                         eccentricity evolution at constant angular momentum.
coordinates (enum)           No          Type of elements to use for modification (Jacobi, barycentric or particle).
                                         See the examples for usage.
============================ =========== ==================================================================

**Particle Parameters**

One can pick and choose which particles have which parameters set.  
For each particle, any unset parameter is ignored.

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
tau_a (double)               No          Semimajor axis exponential growth/damping timescale
tau_e (double)               No          Eccentricity exponential growth/damping timescale
tau_inc (double)             No          Inclination axis exponential growth/damping timescale
tau_Omega (double)           No          Period of linear nodal precession/regression
tau_omega (double)           No          Period of linear apsidal precession/regression
============================ =========== ==================================================================


.. _type_I_migration:

type_I_migration
****************

======================= ===============================================
Authors                 Kajtazi, Kaltrina and D. Petit, C. Antoine
Implementation Paper    `Kajtazi et al 2022 <https://ui.adsabs.harvard.edu/abs/2022arXiv221106181K/abstract>`_.
Based on                `Cresswell & Nelson 2008 <https://ui.adsabs.harvard.edu/abs/2008A%26A...482..677C/abstract>`_, and `Pichierri et al 2018 <https://ui.adsabs.harvard.edu/abs/2018CeMDA.130...54P/abstract>`_.
C example               :ref:`c_example_type_I_migration`
Python example          `TypeIMigration.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/TypeIMigration.ipynb>`_.
======================= ===============================================

This applies Type I migration, damping eccentricity, angular momentum and inclination.
The base of the code is the same as the modified orbital forces one written by D. Tamayo, H. Rein.
It also allows for parameters describing an inner disc edge, modeled using the implementation in inner_disk_edge.c.
Note that this code is not machine independent since power laws were not possible to avoid all together.

**Effect Parameters**

===================================== =========== ==================================================================================================================
Field (C type)                        Required    Description
===================================== =========== ==================================================================================================================
ide_position (double)                 No          The position of the inner disk edge in code units 
ide_width (double)                    No          The disk edge width (planet will stop within ide_width of ide_position)
tIm_surface_density_1 (double)        Yes         Disk surface density at one code unit from the star; used to find the surface density at any distance from the star
tIm_scale_height_1 (double)           Yes         The scale height at one code unit from the star; used to find the aspect ratio at any distance from the star
tIm_surface_density_exponent (double) Yes         Exponent of disk surface density, indicative of the surface density profile of the disk
tIm_flaring_index (double)            Yes         The flaring index; 1 means disk is irradiated by only the stellar flux
===================================== =========== ==================================================================================================================


.. _exponential_migration:

exponential_migration
*********************

======================= ===============================================
Author                   Mohamad Ali-Dib
Implementation Paper    `Ali-Dib et al., 2021 AJ <https://arxiv.org/abs/2104.04271>`_.
Based on                `Hahn & Malhotra 2005 <https://ui.adsabs.harvard.edu/abs/2005AJ....130.2392H/abstract>`_.
C Example               :ref:`c_example_exponential_migration`
Python Example          `ExponentialMigration.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/ExponentialMigration.ipynb>`_.
======================= ===============================================

Continuous velocity kicks leading to exponential change in the object's semimajor axis. 
One of the standard prescriptions often used in Neptune migration & Kuiper Belt formation models.
Does not directly affect the eccentricity or inclination of the object.

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
em_tau_a (double)              Yes          Semimajor axis exponential growth/damping timescale
em_aini (double)               Yes          Object's initial semimajor axis
em_afin (double)               Yes          Object's final semimajor axis
============================ =========== ==================================================================


General Relativity
^^^^^^^^^^^^^^^^^^

.. _lense_thirring:

lense_thirring
**************

======================= ===============================================
Authors                 A. Akmal
Implementation Paper    None
Based on                `Park et al. <https://iopscience.iop.org/article/10.3847/1538-3881/abd414/>`_.
C Example               :ref:`c_example_lense_thirring`
Python Example          `LenseThirring.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/LenseThirring.ipynb>`_.
======================= ===============================================

Adds Lense-Thirring effect due to rotating central body in the simulation. Assumes the source body is particles[0]

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
lt_c (double)                Yes         Speed of light in the units used for the simulation.
============================ =========== ==================================================================

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
I (double)                   Yes         Moment of Inertia of source body 
Omega (reb_vec3d)            Yes         Angular rotation frequency (Omega_x, Omega_y, Omega_z) 
============================ =========== ==================================================================


.. _gr_potential:

gr_potential
************

======================= ===============================================
Authors                 H. Rein, D. Tamayo
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_.
Based on                `Nobili and Roxburgh 1986 <http://labs.adsabs.harvard.edu/adsabs/abs/1986IAUS..114..105N/>`_.
C Example               :ref:`c_example_gr`
Python Example          `GeneralRelativity.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/GeneralRelativity.ipynb>`_.
======================= ===============================================

This is the simplest potential you can use for general relativity.
It assumes that the masses are dominated by a single central body.
It gets the precession right, but gets the mean motion wrong by :math:`\mathcal{O}(GM/ac^2)`.  
It's the fastest option, and because it's not velocity-dependent, it automatically keeps WHFast symplectic.  
Nice if you have a single-star system, don't need to get GR exactly right, and want speed.

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
c (double)                   Yes         Speed of light, needs to be specified in the units used for the simulation.
============================ =========== ==================================================================



.. _gr:

gr
**

======================= ===============================================
Authors                 P. Shi, D. Tamayo, H. Rein
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_.
Based on                `Anderson et al. 1975 <http://labs.adsabs.harvard.edu/adsabs/abs/1975ApJ...200..221A/>`_.
C Example               :ref:`c_example_gr`
Python Example          `GeneralRelativity.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/GeneralRelativity.ipynb>`_.
======================= ===============================================

This assumes that the masses are dominated by a single central body, and should be good enough for most applications with planets orbiting single stars.
It ignores terms that are smaller by of order the mass ratio with the central body.
It gets both the mean motion and precession correct, and will be significantly faster than :ref:`gr_full`, particularly with several bodies.
Adding this effect to several bodies is NOT equivalent to using gr_full.

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
c (double)                   Yes         Speed of light, needs to be specified in the units used for the simulation.
============================ =========== ==================================================================



.. _gr_full:

gr_full
*******

======================= ===============================================
Authors                 P. Shi, H. Rein, D. Tamayo
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_.
Based on                `Newhall et al. 1983 <http://labs.adsabs.harvard.edu/adsabs/abs/1983A%26A...125..150N/>`_.
C Example               :ref:`c_example_gr`
Python Example          `GeneralRelativity.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/GeneralRelativity.ipynb>`_.
======================= ===============================================

This algorithm incorporates the first-order post-newtonian effects from all bodies in the system, and is necessary for multiple massive bodies like stellar binaries.

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
c (double)                   Yes         Speed of light, needs to be specified in the units used for the simulation.
============================ =========== ==================================================================

**Particle Parameters**

*None*

Radiation Forces
^^^^^^^^^^^^^^^^

.. _yarkovsky_effect:

yarkovsky_effect
****************

======================= ===============================================
Authors                 Noah Ferich, D. Tamayo
Implementation Paper    Ferich et al., in prep.
Based on                `Veras et al., 2015 <https://ui.adsabs.harvard.edu/abs/2015MNRAS.451.2814V/abstract>`_, `Veras et al., 2019 <https://ui.adsabs.harvard.edu/abs/2019MNRAS.485..708V/abstract>`_.
C Example               :ref:`c_example_yarkovsky_effect`.
Python Example          `YarkovskyEffect.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/YarkovskyEffect.ipynb>`_.
======================= ===============================================

Adds the accelerations and orbital perturbations created by the Yarkovsky effect onto one or more bodies in the simulation. There are two distinct versions of this effect that can be used: the 'full version' and the 'simple version'. The full version uses the full equations found in Veras et al. (2015) to accurately calculate the Yarkovsky effect on a particle. However, this version slows down simulations and requies a large amount of parameters. For these reasons, the simple version of the effect (based on Veras et al. (2019)) is available. While the magnitude of the acceleration created by the effect will be the same, this version places constant values in a crucial rotation matrix to simplify the push from the Yarkovsky effect on a body. This version is faster and requires less parameters and can be used to get an upper bound on how much the Yarkovsky effect can push an object's orbit inwards or outwards. The lists below describes which parameters are needed for one or both versions of this effect. For more information, please visit the papers and examples linked above.

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
ye_lstar (float)             Yes         Luminosity of sim's star (Required for both versions).
ye_c (float)                 Yes         Speed of light (Required for both versions).
ye_stef_boltz (float)        No          Stefan-Boltzmann constant (Required for full version).
============================ =========== ==================================================================

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
particles[i].r (float)       Yes         Physical radius of a body (Required for both versions).
ye_flag (int)                Yes         0 sets full version of effect. 1 uses simple version with outward migration. -1 uses the simple version with inward migration (see examples and paper).
ye_body_density (float)      Yes         Density of an object (Required for both versions)
ye_rotation_period (float)   No          Rotation period of a spinning object (Required for full version)
ye_albedo (float)            Yes         Albedo of an object (Reuired for both versions)
ye_emissivity (float)        No          Emissivity of an object (Required for full version)
ye_thermal_inertia (float)   No          Thermal inertia of an object (Required for full version)
ye_k (float)                 No          A constant that gets a value between 0 and 1/4 based on the object's rotation - see Veras et al. (2015) for more information on it (Required for full version)
ye_spin_axis_x (float)       No          The x value for the spin axis vector of an object (Required for full version)
ye_spin_axis_y (float)       No          The y value for the spin axis vector of an object (Required for full version)
ye_spin_axis_z (float)       No          The z value for the spin axis vector of an object (Required for full version)
============================ =========== ==================================================================


.. _radiation_forces:

radiation_forces
****************

======================= ===============================================
Authors                 H. Rein, D. Tamayo
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_.
Based on                `Burns et al. 1979 <http://labs.adsabs.harvard.edu/adsabs/abs/1979Icar...40....1B/>`_.
C Example               :ref:`c_example_rad_forces_debris_disk`, :ref:`c_example_rad_forces_circumplanetary`.
Python Example          `Radiation_Forces_Debris_Disk.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/Radiation_Forces_Debris_Disk.ipynb>`_,
                        `Radiation_Forces_Circumplanetary_Dust.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/Radiation_Forces_Circumplanetary_Dust.ipynb>`_.
======================= ===============================================

This applies radiation forces to particles in the simulation.  
It incorporates both radiation pressure and Poynting-Robertson drag.
Only particles whose `beta` parameter is set will feel the radiation.  

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
c (double)                   Yes         Speed of light in the units used for the simulation.
============================ =========== ==================================================================

**Particle Parameters**

If no particles have radiation_source set, effect will assume the particle at index 0 in the particles array is the source.

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
radiation_source (int)       No          Flag identifying the particle as the source of radiation.
beta (float)                 Yes         Ratio of radiation pressure force to gravitational force. Particles without beta set feel no radiation forces.
============================ =========== ==================================================================


Stochastic Forces
^^^^^^^^^^^^^^^^^

.. _stochastic_forces:

stochastic_forces
*****************

======================= ===============================================
Authors                 H. Rein
Based on                `Rein and Papaloizou 2009 <https://ui.adsabs.harvard.edu/abs/2009A%26A...497..595R/abstract>`_.
Implementation Paper    `Rein and Choksi 2022 <https://iopscience.iop.org/article/10.3847/2515-5172/ac6e41>`_.
C Example               :ref:`c_example_stochastic_forces`
Python Example          `StochasticForces.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/StochasticForces.ipynb>`_, `StochasticForcesCartesian.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/StochasticForcesCartesian.ipynb>`_,
======================= ===============================================

This applies stochastic forces to particles in the simulation.  

**Effect Parameters**

None

**Particle Parameters**

All particles which have the field kappa set, will experience stochastic forces.
The particle with index 0 cannot experience stochastic forces.

============================ =========== ==================================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================================
kappa (double)               Yes         Strength of stochastic forces relative to gravity from central object 
tau_kappa (double)           No          Auto-correlation time of stochastic forces. Defaults to orbital period if not set.
                                         The units are relative to the current orbital period.
============================ =========== ==================================================================================


Mass Modifications
^^^^^^^^^^^^^^^^^^

.. _modify_mass:

modify_mass
***********

======================= ===============================================
Authors                 D. Tamayo
Implementation Paper    `Kostov et al., 2016 <https://ui.adsabs.harvard.edu/abs/2016ApJ...832..183K/abstract>`_.
Based on                None
C Example               :ref:`c_example_modify_mass`
Python Example          `ModifyMass.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/ModifyMass.ipynb>`_.
======================= ===============================================

This adds exponential mass growth/loss to individual particles every timestep.
Set particles' ``tau_mass`` parameter to a negative value for mass loss, positive for mass growth.

**Effect Parameters**

*None*

**Particle Parameters**

Only particles with their ``tau_mass`` parameter set will have their masses affected.

============================ =========== =======================================================
Name (C type)                Required    Description
============================ =========== =======================================================
tau_mass (double)            Yes         e-folding mass loss (<0) or growth (>0) timescale    
============================ =========== =======================================================


Tides
^^^^^

.. _tides_spin:

tides_spin
**********

======================= ===============================================
Authors                 Tiger Lu, Hanno Rein, D. Tamayo, Sam Hadden, Rosemary Mardling, Sarah Millholland, Gregory Laughlin
Implementation Paper    `Lu et al., 2023 <https://arxiv.org/abs/2303.00006>`_.
Based on                `Eggleton et al. 1998 <https://ui.adsabs.harvard.edu/abs/1998ApJ...499..853E/abstract>`_.
C Example               :ref:`c_example_tides_spin_pseudo_synchronization`, :ref:`c_example_tides_spin_migration_driven_obliquity_tides`, :ref:`c_example_tides_spin_kozai`.
Python Example          `SpinsIntro.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/SpinsIntro.ipynb>`_, `TidesSpinPseudoSynchronization.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/TidesSpinPseudoSynchronization.ipynb>`_, `TidesSpinEarthMoon.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/TidesSpinEarthMoon.ipynb>`_.
======================= ===============================================

This effect consistently tracks both the spin and orbital evolution of bodies under constant-time lag tides raised on both the primary and on the orbiting bodies.
In all cases, we need to set masses for all the particles that will feel these tidal forces. Particles with only mass are point particles.

Particles are assumed to have structure (i.e - physical extent & distortion from spin) if the following parameters are set: physical radius particles[i].r, potential Love number of degree 2 k2 (Q/(1-Q) in Eggleton 1998), and the spin angular rotation frequency vector Omega.
If we wish to evolve a body's spin components, the fully dimensional moment of inertia I must be set as well. If this parameter is not set, the spin components will be stationary. Note that if the body is a test particle, this is assumed to be the specific moment of inertia.
Finally, if we wish to consider the effects of tides raised on a specific body, we must set the constant time lag tau as well.

For spins that are synchronized with a circular orbit, the constant time lag can be related to the tidal quality factor Q as tau = 1/(2*n*tau), with n the orbital mean motion.
See Lu et. al (in review) and Eggleton et. al (1998) above for discussion.


**Effect Parameters**

None

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
particles[i].r (float)       Yes         Physical radius (required for contribution from tides raised on the body).
k2 (float)                   Yes         Potential Love number of degree 2.
Omega (reb_vec3d)            Yes         Angular rotation frequency (Omega_x, Omega_y, Omega_z)
I (float)                    No          Moment of inertia (for test particles, assumed to be the specific MoI I/m)
tau (float)                  No          Constant time lag. If not set, defaults to 0
============================ =========== ==================================================================


.. _tides_constant_time_lag:

tides_constant_time_lag
***********************

======================= ===============================================
Authors                 Stanley A. Baronett, D. Tamayo, Noah Ferich
Implementation Paper    `Baronett et al., 2022 <https://ui.adsabs.harvard.edu/abs/2022MNRAS.510.6001B/abstract>`_.
Based on                `Hut 1981 <https://ui.adsabs.harvard.edu/#abs/1981A&A....99..126H/abstract>`_, `Bolmont et al., 2015 <https://ui.adsabs.harvard.edu/abs/2015A%26A...583A.116B/abstract>`_.
C Example               :ref:`c_example_tides_constant_time_lag`.
Python Example          `TidesConstantTimeLag.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/TidesConstantTimeLag.ipynb>`_.
======================= ===============================================

This adds constant time lag tidal interactions between orbiting bodies in the simulation and the primary, both from tides raised on the primary and on the other bodies.
In all cases, we need to set masses for all the particles that will feel these tidal forces. After that, we can choose to include tides raised on the primary, on the "planets", or both, by setting the respective bodies' physical radius particles[i].r, k2 (potential Love number of degree 2), constant time lag tau, and rotation rate Omega. See Baronett et al. (2021), Hut (1981), and Bolmont et al. 2015 above.

If tau is not set, it will default to zero and yield the conservative piece of the tidal potential.

**Effect Parameters**

None

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
particles[i].r (float)       Yes         Physical radius (required for contribution from tides raised on the body).
tctl_k2 (float)              Yes         Potential Love number of degree 2.
tctl_tau (float)             No          Constant time lag. If not set will default to 0 and give conservative tidal potential.
OmegaMag (float)             No          Angular rotation frequency. If not set will default to 0.
============================ =========== ==================================================================


Central Force
^^^^^^^^^^^^^

.. _central_force:

central_force
*************

======================= ===============================================
Authors                 D. Tamayo
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_.
Based on                None
C Example               :ref:`c_example_central_force`
Python Example          `CentralForce.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/CentralForce.ipynb>`_.
======================= ===============================================

Adds a general central acceleration of the form a=Acentral*r^gammacentral, outward along the direction from a central particle to the body.
Effect is turned on by adding Acentral and gammacentral parameters to a particle, which will act as the central body for the effect,
and will act on all other particles.

**Effect Parameters**

None

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
Acentral (double)             Yes         Normalization for central acceleration.
gammacentral (double)         Yes         Power index for central acceleration.
============================ =========== ==================================================================


Gravity Fields
^^^^^^^^^^^^^^

.. _gravitational_harmonics:

gravitational_harmonics
***********************

======================= ===============================================
Authors                 M. Broz
Implementation Paper    `Tamayo, Rein, Shi and Hernandez, 2019 <https://ui.adsabs.harvard.edu/abs/2020MNRAS.491.2885T/abstract>`_. 
Based on                None
C Example               :ref:`c_example_j2`
Python Example          `J2.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/J2.ipynb>`_.
======================= ===============================================

Allows the user to add azimuthally symmetric gravitational harmonics (J2, J4) to bodies in the simulation. 
These interact with all other bodies in the simulation (treated as point masses). 
The implementation allows the user to specify an arbitrary spin axis orientation for each oblate body, which defines the axis of symmetry.
This is specified through the angular rotation rate vector Omega.
The rotation rate Omega is not currently used other than to specify the spin axis orientation.
In particular, the current implementation applies the appropriate torque from the body's oblateness to the orbits of all the other planets, but does not account for the equal and opposite torque on the body's spin angular momentum.
The bodies spins therefore remain constant in the current implementation.
This is a good approximation in the limit where the bodies' spin angular momenta are much greater than the orbital angular momenta involved.

**Effect Parameters**

None

**Particle Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
J2 (double)                  No          J2 coefficient
J4 (double)                  No          J4 coefficient
R_eq (double)                No          Equatorial radius of nonspherical body used for calculating Jn harmonics
Omega (reb_vec3d)            No          Angular rotation frequency (Omega_x, Omega_y, Omega_z)
============================ =========== ==================================================================


Gas Effects
^^^^^^^^^^^
.. _gas_dynamical_friction:

gas_dynamical_friction
**********************

======================= ===============================================
Authors                 A. Generozov, H. Perets
Implementation Paper    `Generozov and Perets 2022 <https://arxiv.org/abs/2212.11301>`_
Based on                `Ostriker 1999 (with simplifications) <https://ui.adsabs.harvard.edu/abs/1999ApJ...513..252O/abstract>`_, `Just et al 2012 <https://ui.adsabs.harvard.edu/abs/2012ApJ...758...51J/abstract>`_.
C Example               :ref:`c_example_gas_dynamical_friction`
Python Example          `GasDynamicalFriction.ipynb <https://github.com/dtamayo/reboundx/blob/master/ipython_examples/GasDynamicalFriction.ipynb>`_
                       

======================= ===============================================


**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
rhog (double)                Yes         Normalization of density. Density in the disk midplane is rhog*r^alpha_rhog
alpha_rhog (double)          Yes         Power-law slope of the power-law density profile.
cs (double)                  Yes         Normalization of the sound speed. Sound speed has profile cs*r^alpha_cs
alpha_cs (double)            Yes         Power-law slope of the sound speed
xmin (double)                Yes         Dimensionless parameter that determines the Coulomb logarithm (ln(L) =log (1/xmin))
hr (double)                  Yes         Aspect ratio of the disk
Qd (double)                  Yes         Prefactor for geometric drag
============================ =========== ==================================================================


**Particle Parameters**

None.


.. _gas_damping_timescale:

gas_damping_timescale
*********************

======================= ===============================================
Authors                 Phoebe Sandhaus
Implementation Paper    `Sandhaus et al. in prep`
Based on                `Dawson et al. 2016 <https://ui.adsabs.harvard.edu/abs/2016ApJ...822...54D/abstract>; Kominami & Ida 2002 <https://ui.adsabs.harvard.edu/abs/2002Icar..157...43K/abstract>`_
C Example               :ref:`c_example_gas_damping_timescale`
Python Example          `GasDampingTimescale.ipynb <https://github.com/PhoebeSandhaus/reboundx_gas_damping/tree/main/ipython_examples/GasDampingTimescale.ipynb>`_
======================= ===============================================

This updates particles' positions and velocities between timesteps by first calculating a damping timescale for each individual particle, and then applying the timescale to damp both the eccentricity and inclination of the particle. Note: The timescale of damping should be much greater than a particle's orbital period. The damping force should also be small as compared to the gravitational forces on the particle.

**Effect Parameters**

============================ =========== ==================================================================
Field (C type)               Required    Description
============================ =========== ==================================================================
None                         -           -
============================ =========== ==================================================================

**Particle Parameters**

============================ =========== ==============================================================================
Field (C type)               Required    Description
============================ =========== ==============================================================================
d_factor (double)            Yes         Depletion factor d in Equation 16 from Dawson et al. 2016; d=1 corresponds roughly to the full minimum mass solar nebula with Sigma_gas (surface gas density) = 1700 g cm^-2 at 1 AU [for d=1]; d>1 corresponds to a more depleted nebula
cs_coeff (double)            Yes         Sound speed coefficient; Changing the value will change assumed units; Example: If you are using the units: AU, M_sun, yr -->  cs_coeff = 0.272  # AU^(3/4) yr^-1
tau_coeff (double)           Yes         Timescale coefficient;   Changing the value will change assumed units; Example: If you are using the units: AU, M_sun, yr --> tau_coeff = 0.003  #  yr AU^-2
============================ =========== ==============================================================================


