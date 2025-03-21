.. index:: compute

compute command
===============

Syntax
""""""

.. code-block:: LAMMPS

   compute ID group-ID style args

* ID = user-assigned name for the computation
* group-ID = ID of the group of atoms to perform the computation on
* style = one of a list of possible style names (see below)
* args = arguments used by a particular style

Examples
""""""""

.. code-block:: LAMMPS

   compute 1 all temp
   compute newtemp flow temp/partial 1 1 0
   compute 3 all ke/atom

Description
"""""""""""

Define a diagnostic computation that will be performed on a group of
atoms.  Quantities calculated by a compute are instantaneous values,
meaning they are calculated from information about atoms on the
current timestep or iteration, though internally a compute may store
some information about a previous state of the system.  Defining a
compute does not perform the computation.  Instead computes are
invoked by other LAMMPS commands as needed (e.g., to calculate a
temperature needed for a thermostat fix or to generate thermodynamic
or dump file output).  See the :doc:`Howto output <Howto_output>` page
for a summary of various LAMMPS output options, many of which involve
computes.

The ID of a compute can only contain alphanumeric characters and
underscores.

----------

Computes calculate and store any of four *styles* of quantities:
global, per-atom, local, or per-grid.

A global quantity is one or more system-wide values, e.g. the
temperature of the system.  A per-atom quantity is one or more values
per atom, e.g. the kinetic energy of each atom.  Per-atom values are
set to 0.0 for atoms not in the specified compute group.  Local
quantities are calculated by each processor based on the atoms it
owns, but there may be zero or more per atom, e.g. a list of bond
distances.  Per-grid quantities are calculated on a regular 2d or 3d
grid which overlays a 2d or 3d simulation domain.  The grid points and
the data they store are distributed across processors; each processor
owns the grid points which fall within its subdomain.

As a general rule of thumb, computes that produce per-atom quantities
have the word "atom" at the end of their style, e.g. *ke/atom*\ .
Computes that produce local quantities have the word "local" at the
end of their style, e.g. *bond/local*\ .  Computes that produce
per-grid quantities have the word "grid" at the end of their style,
e.g. *property/grid*\ .  And styles with neither "atom" or "local" or
"grid" at the end of their style name produce global quantities.

Global, per-atom, local, and per-grid quantities can also be of three
*kinds*: a single scalar value (global only), a vector of values, or a
2d array of values.  For per-atom, local, and per-grid quantities, a
"vector" means a single value for each atom, each local entity
(e.g. bond), or grid cell.  Likewise an "array", means multiple values
for each atom, each local entity, or each grid cell.

Note that a single compute can produce any combination of global,
per-atom, local, or per-grid values.  Likewise it can produce any
combination of scalar, vector, or array output for each style.  The
exception is that for per-atom, local, and per-grid output, either a
vector or array can be produced, but not both.  The doc page for each
compute explains the values it produces.

When a compute output is accessed by another input script command it
is referenced via the following bracket notation, where ID is the ID
of the compute:

+-------------+--------------------------------------------+
| c_ID        | entire scalar, vector, or array            |
+-------------+--------------------------------------------+
| c_ID[I]     | one element of vector, one column of array |
+-------------+--------------------------------------------+
| c_ID[I][J]  | one element of array                       |
+-------------+--------------------------------------------+

In other words, using one bracket reduces the dimension of the
quantity once (vector :math:`\to` scalar, array :math:`\to` vector).
Using two brackets reduces the dimension twice (array :math:`\to`
scalar).  Thus, for example, a command that uses global scalar compute
values as input can also process elements of a vector or array.
Depending on the command, this can either be done directly using the
syntax in the table, or by first defining a :doc:`variable <variable>`
of the appropriate style to store the quantity, then using the
variable as an input to the command.

Note that commands and :doc:`variables <variable>` which take compute
outputs as input typically do not allow for all styles and kinds of
data (e.g., a command may require global but not per-atom values, or
it may require a vector of values, not a scalar).  This means there is
typically no ambiguity about referring to a compute output as c_ID
even if it produces, for example, both a scalar and vector.  The doc
pages for various commands explain the details, including how any
ambiguities are resolved.

----------

In LAMMPS, the values generated by a compute can be used in several
ways:

* The results of computes that calculate a global temperature or
  pressure can be used by fixes that do thermostatting or barostatting
  or when atom velocities are created.
* Global values can be output via the :doc:`thermo_style custom <thermo_style>` or :doc:`fix ave/time <fix_ave_time>` command.
  Or the values can be referenced in a :doc:`variable equal <variable>` or
  :doc:`variable atom <variable>` command.
* Per-atom values can be output via the :doc:`dump custom <dump>` command.
  Or they can be time-averaged via the :doc:`fix ave/atom <fix_ave_atom>`
  command or reduced by the :doc:`compute reduce <compute_reduce>`
  command.  Or the per-atom values can be referenced in an :doc:`atom-style variable <variable>`.
* Local values can be reduced by the :doc:`compute reduce <compute_reduce>` command, or histogrammed by the :doc:`fix ave/histo <fix_ave_histo>` command, or output by the :doc:`dump local <dump>` command.

The results of computes that calculate global quantities can be either
"intensive" or "extensive" values.  Intensive means the value is
independent of the number of atoms in the simulation
(e.g., temperature).  Extensive means the value scales with the number of
atoms in the simulation (e.g., total rotational kinetic energy).
:doc:`Thermodynamic output <thermo_style>` will normalize extensive
values by the number of atoms in the system, depending on the
"thermo_modify norm" setting.  It will not normalize intensive values.
If a compute value is accessed in another way (e.g., by a
:doc:`variable <variable>`), you may want to know whether it is an
intensive or extensive value.  See the page for individual
computes for further info.

----------

LAMMPS creates its own computes internally for thermodynamic output.
Three computes are always created, named "thermo_temp",
"thermo_press", and "thermo_pe", as if these commands had been invoked
in the input script:

.. code-block:: LAMMPS

   compute thermo_temp all temp
   compute thermo_press all pressure thermo_temp
   compute thermo_pe all pe

Additional computes for other quantities are created if the thermo
style requires it.  See the documentation for the
:doc:`thermo_style <thermo_style>` command.

Fixes that calculate temperature or pressure, i.e. for thermostatting
or barostatting, may also create computes.  These are discussed in the
documentation for specific :doc:`fix <fix>` commands.

In all these cases, the default computes LAMMPS creates can be
replaced by computes defined by the user in the input script, as
described by the :doc:`thermo_modify <thermo_modify>` and :doc:`fix modify <fix_modify>` commands.

Properties of either a default or user-defined compute can be modified
via the :doc:`compute_modify <compute_modify>` command.

Computes can be deleted with the :doc:`uncompute <uncompute>` command.

Code for new computes can be added to LAMMPS; see the
:doc:`Modify <Modify>` page for details.  The results of their
calculations accessed in the various ways described above.

----------

Each compute style has its own page which describes its arguments
and what it does.  Here is an alphabetic list of compute styles
available in LAMMPS.  They are also listed in more compact form on the
:doc:`Commands compute <Commands_compute>` doc page.

There are also additional accelerated compute styles included in the
LAMMPS distribution for faster performance on CPUs, GPUs, and KNLs.
The individual style names on the :doc:`Commands compute <Commands_compute>` page are followed by one or more of
(g,i,k,o,t) to indicate which accelerated styles exist.

* :doc:`ackland/atom <compute_ackland_atom>` - determines the local lattice structure based on the Ackland formulation
* :doc:`adf <compute_adf>` - angular distribution function of triples of atoms
* :doc:`aggregate/atom <compute_cluster_atom>` - aggregate ID for each atom
* :doc:`angle <compute_angle>` - energy of each angle sub-style
* :doc:`angle/local <compute_angle_local>` - theta and energy of each angle
* :doc:`angmom/chunk <compute_angmom_chunk>` - angular momentum for each chunk
* :doc:`ave/sphere/atom <compute_ave_sphere_atom>` - compute local density and temperature around each atom
* :doc:`basal/atom <compute_basal_atom>` - calculates the hexagonal close-packed "c" lattice vector of each atom
* :doc:`body/local <compute_body_local>` - attributes of body sub-particles
* :doc:`bond <compute_bond>` - energy of each bond sub-style
* :doc:`bond/local <compute_bond_local>` - distance and energy of each bond
* :doc:`born/matrix <compute_born_matrix>` - second derivative or potential with respect to strain
* :doc:`centro/atom <compute_centro_atom>` - centro-symmetry parameter for each atom
* :doc:`centroid/stress/atom <compute_stress_atom>` - centroid based stress tensor for each atom
* :doc:`chunk/atom <compute_chunk_atom>` - assign chunk IDs to each atom
* :doc:`chunk/spread/atom <compute_chunk_spread_atom>` - spreads chunk values to each atom in chunk
* :doc:`cluster/atom <compute_cluster_atom>` - cluster ID for each atom
* :doc:`cna/atom <compute_cna_atom>` - common neighbor analysis (CNA) for each atom
* :doc:`cnp/atom <compute_cnp_atom>` - common neighborhood parameter (CNP) for each atom
* :doc:`com <compute_com>` - center of mass of group of atoms
* :doc:`com/chunk <compute_com_chunk>` - center of mass for each chunk
* :doc:`contact/atom <compute_contact_atom>` - contact count for each spherical particle
* :doc:`coord/atom <compute_coord_atom>` - coordination number for each atom
* :doc:`count/type <compute_count_type>` - count of atoms or bonds by type
* :doc:`damage/atom <compute_damage_atom>` - Peridynamic damage for each atom
* :doc:`dihedral <compute_dihedral>` - energy of each dihedral sub-style
* :doc:`dihedral/local <compute_dihedral_local>` - angle of each dihedral
* :doc:`dilatation/atom <compute_dilatation_atom>` - Peridynamic dilatation for each atom
* :doc:`dipole <compute_dipole>` - dipole vector and total dipole
* :doc:`dipole/chunk <compute_dipole_chunk>` - dipole vector and total dipole for each chunk
* :doc:`dipole/tip4p <compute_dipole>` - dipole vector and total dipole with TIP4P pair style
* :doc:`dipole/tip4p/chunk <compute_dipole_chunk>` - dipole vector and total dipole for each chunk with TIP4P pair style
* :doc:`displace/atom <compute_displace_atom>` - displacement of each atom
* :doc:`dpd <compute_dpd>` - total values of internal conductive energy, internal mechanical energy, chemical energy, and harmonic average of internal temperature
* :doc:`dpd/atom <compute_dpd_atom>` - per-particle values of internal conductive energy, internal mechanical energy, chemical energy, and internal temperature
* :doc:`edpd/temp/atom <compute_edpd_temp_atom>` - per-atom temperature for each eDPD particle in a group
* :doc:`efield/atom <compute_efield_atom>` - electric field at each atom
* :doc:`efield/wolf/atom <compute_efield_wolf_atom>` - electric field at each atom
* :doc:`entropy/atom <compute_entropy_atom>` - pair entropy fingerprint of each atom
* :doc:`erotate/asphere <compute_erotate_asphere>` - rotational energy of aspherical particles
* :doc:`erotate/rigid <compute_erotate_rigid>` - rotational energy of rigid bodies
* :doc:`erotate/sphere <compute_erotate_sphere>` - rotational energy of spherical particles
* :doc:`erotate/sphere/atom <compute_erotate_sphere_atom>` - rotational energy for each spherical particle
* :doc:`event/displace <compute_event_displace>` - detect event on atom displacement
* :doc:`fabric <compute_fabric>` - calculates fabric tensors from pair interactions
* :doc:`fep <compute_fep>` - compute free energies for alchemical transformation from perturbation theory
* :doc:`fep/ta <compute_fep_ta>` - compute free energies for a test area perturbation
* :doc:`force/tally <compute_tally>` - force between two groups of atoms via the tally callback mechanism
* :doc:`fragment/atom <compute_cluster_atom>` - fragment ID for each atom
* :doc:`gaussian/grid/local <compute_gaussian_grid_local>` - local array of Gaussian atomic contributions on a regular grid
* :doc:`global/atom <compute_global_atom>` - assign global values to each atom from arrays of global values
* :doc:`group/group <compute_group_group>` - energy/force between two groups of atoms
* :doc:`gyration <compute_gyration>` - radius of gyration of group of atoms
* :doc:`gyration/chunk <compute_gyration_chunk>` - radius of gyration for each chunk
* :doc:`gyration/shape <compute_gyration_shape>` - shape parameters from gyration tensor
* :doc:`gyration/shape/chunk <compute_gyration_shape_chunk>` - shape parameters from gyration tensor for each chunk
* :doc:`heat/flux <compute_heat_flux>` - heat flux through a group of atoms
* :doc:`heat/flux/tally <compute_tally>` - heat flux through a group of atoms via the tally callback mechanism
* :doc:`heat/flux/virial/tally <compute_tally>` - virial heat flux between two groups via the tally callback mechanism
* :doc:`hexorder/atom <compute_hexorder_atom>` - bond orientational order parameter q6
* :doc:`hma <compute_hma>` - harmonically mapped averaging for atomic crystals
* :doc:`improper <compute_improper>` - energy of each improper sub-style
* :doc:`improper/local <compute_improper_local>` - angle of each improper
* :doc:`inertia/chunk <compute_inertia_chunk>` - inertia tensor for each chunk
* :doc:`ke <compute_ke>` - translational kinetic energy
* :doc:`ke/atom <compute_ke_atom>` - kinetic energy for each atom
* :doc:`ke/atom/eff <compute_ke_atom_eff>` - per-atom translational and radial kinetic energy in the electron force field model
* :doc:`ke/eff <compute_ke_eff>` - kinetic energy of a group of nuclei and electrons in the electron force field model
* :doc:`ke/rigid <compute_ke_rigid>` - translational kinetic energy of rigid bodies
* :doc:`composition/atom <compute_composition_atom>` - local composition for each atom
* :doc:`mliap <compute_mliap>` - gradients of energy and forces with respect to model parameters and related quantities for training machine learning interatomic potentials
* :doc:`momentum <compute_momentum>` - translational momentum
* :doc:`msd <compute_msd>` - mean-squared displacement of group of atoms
* :doc:`msd/chunk <compute_msd_chunk>` - mean-squared displacement for each chunk
* :doc:`msd/nongauss <compute_msd_nongauss>` - MSD and non-Gaussian parameter of group of atoms
* :doc:`nbond/atom <compute_nbond_atom>` - calculates number of bonds per atom
* :doc:`omega/chunk <compute_omega_chunk>` - angular velocity for each chunk
* :doc:`orientorder/atom <compute_orientorder_atom>` - Steinhardt bond orientational order parameters Ql
* :doc:`pace <compute_pace>` - atomic cluster expansion descriptors and related quantities
* :doc:`pair <compute_pair>` - values computed by a pair style
* :doc:`pair/local <compute_pair_local>` - distance/energy/force of each pairwise interaction
* :doc:`pe <compute_pe>` - potential energy
* :doc:`pe/atom <compute_pe_atom>` - potential energy for each atom
* :doc:`pe/mol/tally <compute_tally>` - potential energy between two groups of atoms separated into intermolecular and intramolecular components via the tally callback mechanism
* :doc:`pe/tally <compute_tally>` - potential energy between two groups of atoms via the tally callback mechanism
* :doc:`plasticity/atom <compute_plasticity_atom>` - Peridynamic plasticity for each atom
* :doc:`pod/atom <compute_pod_atom>` - POD descriptors for each atom
* :doc:`podd/atom <compute_pod_atom>` - derivative of POD descriptors for each atom
* :doc:`pod/local <compute_pod_atom>` - local POD descriptors and their derivatives
* :doc:`pod/global <compute_pod_atom>` - global POD descriptors and their derivatives
* :doc:`pressure <compute_pressure>` - total pressure and pressure tensor
* :doc:`pressure/alchemy <compute_pressure_alchemy>` - mixed system total pressure and pressure tensor for :doc:`fix alchemy <fix_alchemy>` runs
* :doc:`pressure/uef <compute_pressure_uef>` - pressure tensor in the reference frame of an applied flow field
* :doc:`property/atom <compute_property_atom>` - convert atom attributes to per-atom vectors/arrays
* :doc:`property/chunk <compute_property_chunk>` - extract various per-chunk attributes
* :doc:`property/grid <compute_property_grid>` - convert per-grid attributes to per-grid vectors/arrays
* :doc:`property/local <compute_property_local>` - convert local attributes to local vectors/arrays
* :doc:`ptm/atom <compute_ptm_atom>` - determines the local lattice structure based on the Polyhedral Template Matching method
* :doc:`rattlers/atom <compute_rattlers_atom>` - identify under-coordinated rattler atoms
* :doc:`rdf <compute_rdf>` - radial distribution function :math:`g(r)` histogram of group of atoms
* :doc:`reaxff/atom <compute_reaxff_atom>` - extract ReaxFF bond information
* :doc:`reduce <compute_reduce>` - combine per-atom quantities into a single global value
* :doc:`reduce/chunk <compute_reduce_chunk>` - reduce per-atom quantities within each chunk
* :doc:`reduce/region <compute_reduce>` - same as compute reduce, within a region
* :doc:`rheo/property/atom <compute_rheo_property_atom>` - convert atom attributes in RHEO package to per-atom vectors/arrays
* :doc:`rigid/local <compute_rigid_local>` - extract rigid body attributes
* :doc:`saed <compute_saed>` - electron diffraction intensity on a mesh of reciprocal lattice nodes
* :doc:`slcsa/atom <compute_slcsa_atom>` - perform Supervised Learning Crystal Structure Analysis (SL-CSA)
* :doc:`slice <compute_slice>` - extract values from global vector or array
* :doc:`smd/contact/radius <compute_smd_contact_radius>` - contact radius for Smooth Mach Dynamics
* :doc:`smd/damage <compute_smd_damage>` - damage status of SPH particles in Smooth Mach Dynamics
* :doc:`smd/hourglass/error <compute_smd_hourglass_error>` - error associated with approximated relative separation in Smooth Mach Dynamics
* :doc:`smd/internal/energy <compute_smd_internal_energy>` - per-particle enthalpy in Smooth Mach Dynamics
* :doc:`smd/plastic/strain <compute_smd_plastic_strain>` - equivalent plastic strain per particle in Smooth Mach Dynamics
* :doc:`smd/plastic/strain/rate <compute_smd_plastic_strain_rate>` - time rate of the equivalent plastic strain in Smooth Mach Dynamics
* :doc:`smd/rho <compute_smd_rho>` - per-particle mass density in Smooth Mach Dynamics
* :doc:`smd/tlsph/defgrad <compute_smd_tlsph_defgrad>` - deformation gradient in Smooth Mach Dynamics
* :doc:`smd/tlsph/dt <compute_smd_tlsph_dt>` - CFL-stable time increment per particle in Smooth Mach Dynamics
* :doc:`smd/tlsph/num/neighs <compute_smd_tlsph_num_neighs>` - number of particles inside the smoothing kernel radius for Smooth Mach Dynamics
* :doc:`smd/tlsph/shape <compute_smd_tlsph_shape>` - current shape of the volume of a particle for Smooth Mach Dynamics
* :doc:`smd/tlsph/strain <compute_smd_tlsph_strain>` - Green--Lagrange strain tensor for Smooth Mach Dynamics
* :doc:`smd/tlsph/strain/rate <compute_smd_tlsph_strain_rate>` - rate of strain for Smooth Mach Dynamics
* :doc:`smd/tlsph/stress <compute_smd_tlsph_stress>` - per-particle Cauchy stress tensor for SPH particles
* :doc:`smd/triangle/vertices <compute_smd_triangle_vertices>` - coordinates of vertices corresponding to the triangle elements of a mesh for Smooth Mach Dynamics
* :doc:`smd/ulsph/effm <compute_smd_ulsph_effm>` - per-particle effective shear modulus
* :doc:`smd/ulsph/num/neighs <compute_smd_ulsph_num_neighs>` - number of neighbor particles inside the smoothing kernel radius for Smooth Mach Dynamics
* :doc:`smd/ulsph/strain <compute_smd_ulsph_strain>` - logarithmic strain tensor for Smooth Mach Dynamics
* :doc:`smd/ulsph/strain/rate <compute_smd_ulsph_strain_rate>` - logarithmic strain rate for Smooth Mach Dynamics
* :doc:`smd/ulsph/stress <compute_smd_ulsph_stress>` - per-particle Cauchy stress tensor and von Mises equivalent stress in Smooth Mach Dynamics
* :doc:`smd/vol <compute_smd_vol>` - per-particle volumes and their sum in Smooth Mach Dynamics
* :doc:`snap <compute_sna_atom>` - gradients of SNAP energy and forces with respect to linear coefficients and related quantities for fitting SNAP potentials
* :doc:`sna/atom <compute_sna_atom>` - bispectrum components for each atom
* :doc:`sna/grid <compute_sna_atom>` - global array of bispectrum components on a regular grid
* :doc:`sna/grid/local <compute_sna_atom>` - local array of bispectrum components on a regular grid
* :doc:`snad/atom <compute_sna_atom>` - derivative of bispectrum components for each atom
* :doc:`snav/atom <compute_sna_atom>` - virial contribution from bispectrum components for each atom
* :doc:`sph/e/atom <compute_sph_e_atom>` - per-atom internal energy of Smooth-Particle Hydrodynamics atoms
* :doc:`sph/rho/atom <compute_sph_rho_atom>` - per-atom density of Smooth-Particle Hydrodynamics atoms
* :doc:`sph/t/atom <compute_sph_t_atom>` - per-atom internal temperature of Smooth-Particle Hydrodynamics atoms
* :doc:`spin <compute_spin>` - magnetic quantities for a system of atoms having spins
* :doc:`stress/atom <compute_stress_atom>` - stress tensor for each atom
* :doc:`stress/cartesian <compute_stress_cartesian>` - stress tensor in cartesian coordinates
* :doc:`stress/cylinder <compute_stress_curvilinear>` - stress tensor in cylindrical coordinates
* :doc:`stress/mop <compute_stress_mop>` - normal components of the local stress tensor using the method of planes
* :doc:`stress/mop/profile <compute_stress_mop>` - profile of the normal components of the local stress tensor using the method of planes
* :doc:`stress/spherical <compute_stress_curvilinear>` - stress tensor in spherical coordinates
* :doc:`stress/tally <compute_tally>` - stress between two groups of atoms via the tally callback mechanism
* :doc:`tdpd/cc/atom <compute_tdpd_cc_atom>` - per-atom chemical concentration of a specified species for each tDPD particle
* :doc:`temp <compute_temp>` - temperature of group of atoms
* :doc:`temp/asphere <compute_temp_asphere>` - temperature of aspherical particles
* :doc:`temp/body <compute_temp_body>` - temperature of body particles
* :doc:`temp/chunk <compute_temp_chunk>` - temperature of each chunk
* :doc:`temp/com <compute_temp_com>` - temperature after subtracting center-of-mass velocity
* :doc:`temp/cs <compute_temp_cs>` - temperature based on the center-of-mass velocity of atom pairs that are bonded to each other
* :doc:`temp/deform <compute_temp_deform>` - temperature excluding box deformation velocity
* :doc:`temp/deform/eff <compute_temp_deform_eff>` - temperature excluding box deformation velocity in the electron force field model
* :doc:`temp/drude <compute_temp_drude>` - temperature of Core--Drude pairs
* :doc:`temp/eff <compute_temp_eff>` - temperature of a group of nuclei and electrons in the electron force field model
* :doc:`temp/partial <compute_temp_partial>` - temperature excluding one or more dimensions of velocity
* :doc:`temp/profile <compute_temp_profile>` - temperature excluding a binned velocity profile
* :doc:`temp/ramp <compute_temp_ramp>` - temperature excluding ramped velocity component
* :doc:`temp/region <compute_temp_region>` - temperature of a region of atoms
* :doc:`temp/region/eff <compute_temp_region_eff>` - temperature of a region of nuclei and electrons in the electron force field model
* :doc:`temp/rotate <compute_temp_rotate>` - temperature of a group of atoms after subtracting out their center-of-mass and angular velocities
* :doc:`temp/sphere <compute_temp_sphere>` - temperature of spherical particles
* :doc:`temp/uef <compute_temp_uef>` - kinetic energy tensor in the reference frame of an applied flow field
* :doc:`ti <compute_ti>` - thermodynamic integration free energy values
* :doc:`torque/chunk <compute_torque_chunk>` - torque applied on each chunk
* :doc:`vacf <compute_vacf>` - velocity auto-correlation function of group of atoms
* :doc:`vacf/chunk <compute_vacf_chunk>` - velocity auto-correlation for the center of mass velocities of chunks of atoms
* :doc:`vcm/chunk <compute_vcm_chunk>` - velocity of center-of-mass for each chunk
* :doc:`viscosity/cos <compute_viscosity_cos>` - velocity profile under cosine-shaped acceleration
* :doc:`voronoi/atom <compute_voronoi_atom>` - Voronoi volume and neighbors for each atom
* :doc:`xrd <compute_xrd>` - X-ray diffraction intensity on a mesh of reciprocal lattice nodes

Restrictions
""""""""""""

none

Related commands
""""""""""""""""

:doc:`uncompute <uncompute>`, :doc:`compute_modify <compute_modify>`,
:doc:`fix ave/atom <fix_ave_atom>`, :doc:`fix ave/time <fix_ave_time>`,
:doc:`fix ave/histo <fix_ave_histo>`

Default
"""""""

none
