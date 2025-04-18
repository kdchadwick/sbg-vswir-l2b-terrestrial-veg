# SBG VSWIR Terrestrial Vegetation Equivalent Water Thickness Product - Algorithm Theoretical Basis Document (ATBD)**

*K.D. Chadwick**<sup>1</sup>

<sup>1</sup>Jet Propulsion Laboratory, California Institute of Technology

Corresponding author: K. Dana Chadwick (dana.chadwick@jpl.nasa.gov)
(TBD) 

**Key Points:**
   1. Please note that this ATBD will be updated on an ongoing basis as the SBG VSWIR project progresses. This is intended to be a place where the community can find the most up-to-date information on the current plans for algorithm development and offer contributions.
   2. The canopy equivalent water thickness (EWT) demonstration products are baselined to be generated using Bohn et. al 2020.  

**Version:** 1.0

**Release Date:** TBD

**DOI:** TBD

## Abstract

The SBG-VSWIR mission level 2+ processing for terrestrial vegetation will estimate canopy water content (CWC) via an equivalent water thickness (EWT) estimation.

## Plain Language Summary
This document describes the strategy for producing terrestrial vegetation EWT gridded products that will be produced by the SBG-VSWIR mission. The terrestrial vegetation products will be generated for pixels that are determined through the unmixing products to contain sufficient vegetation cover to warrant estimation. Error will be estimated for each product to the best ability to assess. Here we outline the strategy for scaling terrestrial vegetation EWT products.

### Keywords: hyperspectral imaging, imaging spectroscopy, equivilant water thickness, canopy water content, terrestrial

## 1 Version Description

This is Version 1.0 of the SBG VSWIR terrestrial vegetation algorithm equivalent water thickness.
## 2 Introduction

Following on recommendations by the National Academies of Science, Engineering and Medicine in their 2018 Decadal Survey on Earth Sciences,
the National Aeronautics and Space Administration (NASA) selected the Surface Biology and Geology (SBG) mission for implementation as part of  
its Earth System Observatory (ESO).  The ESO is a series of missions that will launch in the late 2020s and early 2030s to observe different 
aspects of the Earth System.  SBG in particular aims to measure properties of the Earth’s surface composition and ecology. 
It is comprised of two platforms: a wide-swath thermal infrared instrument (TIR) on a free-flying spacecraft in a polar orbit with an afternoon 
equatorial crossing time; and a separate spacecraft carrying a wide-swath solar reflectance imaging spectrometer operating in the Visible Shortwave
 Infrared (VSWIR), with a morning equatorial crossing time.

 
This document focuses on the second platform, SBG-VSWIR.  SBG-VSWIR will measure the solar reflected range at approximately 380-2500 nm at 10 nm spectral sampling, over a 180 km swath with 30 m spatial sampling.  This measurement enables repeat coverage of any location on Earth every 16 days.  The visible-shortwave infrared range is sensitive to diverse physical processes in the Earth’s surface and atmosphere (Cawse-Nicholson 2021). It provides Earth-system-scale measurements relevant to: terrestrial vegetation and biodiversity, including plant functinoal traits and canopy water content.

This Algorithm Theoretical Basis Document describes the algorithm used for terrestrial vegetation equivilent water thickness. Equivalent Water Thickness (EWT) is the predicted thickness or absorption path length in centimeters (cm) of water that would be required to yield the observed spectra. For vegetation, this is equivalent to canopy water content (CWC) in g/cm^2, since one cm^3 of water has a mass of 1g.

<figure>
    <img src="fig/nominal_ewt.PNG" alt="Figure 1: Nominal EWT workflow" width="500">
    <figcaption style="font-style: italic; margin-bottom: 40px;"> Figure 1: Nominal EWT workflow. </figcaption>
</figure> 
 

## 3 Context/Background

### 3.1 Historical Perspective

Equivalent water thickness (EWT) has been critical to advancing udnerstanding of terrestrial vegetation status through remote sensing.
Early use of imaging spectrometers showed the sensitivity of the shortwave infared (SWIR) wavelengths of 950 - 2500nm to water absortion  (Tucker, 1980) and they became esstential for quanitfying water status and drought stress in vegetation. 
The developement of airborne Imaging spectrometers such as NASA's AVIRIS, provided more spectral detail, allowing for mapping of CWC across landscaps.
EWT is important for informing models of photosynthesis, transpiration, and plant responses to climate stressors, influencing studies on ecosystem health, drought monitoring, and wildfire risk (Ustin et al., 2004; Gao, 1996). For example, the work of Cheng et al. (2020) highlighted the ability of hyperspectral data to track drought-induced vegetation stress more precisely than traditional broadband indices. Additionally, advances in radiative transfer modeling allowed for better understanding of how CWC influences hydrological processes, particularly regarding how water is retained and cycled within the canopy.


### 3.2 Additional Information

## 4 Algorithm Description

### 4.1 Scientific Theory
CWC estimation relies on the principle that water exhibits strong wavelength-dependent absorption characteristics in the near-infrared (NIR) range Green et al. (2006)​. When sunlight interacts with plant canopies, part of the light absorbed by the water in the leaves, while part of it reflected back into the atmosphere. By measuring the surface reflectance at the wavelengths corresponding to narrow NIR features, it becomes possible to infer the water content present in the canopy.
Specifically, the Beer-Lambert law is used to model the path length of photon absorption as light interacts with liquid water. The absorption feature of liquid water, particularly between 850 and 1100 nm, allows for the derivation of the path length and thus the estimation of water content. 

Following Bohn et al. 2020 surface reflectance spectra obtained from remote sensing instruments are analyzed  using a least squares inversion technique. 
This process minimizes the difference between the measured reflectance 
and a model based on the Beer-Lambert law, iteratively adjusting the path length of water absorption to fit the reflectance data. 
By leveraging laboratory measurements of water's refractive index, the model can accurately quantify the absorption coefficients of water across different wavelengths.


#### 4.1.2 Scientific theory assumptions
Assumptions in the scientific theory include 
 * Homogenetiy of the canopy 
 * Minimal Atmospheric interference through accurate atmopsheric correction that minimizes the effect of aersols, gases and other atmospheric conditiosn on surface reflectance measurements 

### 4.2 Mathematical Theory

The mathematical foundation for estimating equivilant water thickness is based on the Beer-Lambert law. This model assumes that the amount of light absorbed is exponentially related to the path length of the light through the absorbing material (in this case, liquid water within the canopy) and the material’s absorption coefficient. 

The surface reflectance $\rho_s$ at any given wavelength $\lambda$ is modeled to account for the absorption of 
light by liquid water and ice in snow (Bohn et al. 2020). [View equation code](../algorithms/EWT.py#L32-L50) :

$$
\rho_{(s,\lambda)} = (a+b\lambda) e^{(-d_wa_{w,\lambda} - d_ia_{i,\lambda} )}
$$

Where:
- $a$ and $b$ are offset and slope of the linear reflectance continuum 
- $a_w$ and $a_i$ are the wavelength dependent absorption coeffiecnts 
    of liquid water and ice, respectively.
- $d_w$ and $d_i$ are the liquid water and ice path lengths, respectively. 
And in the case of canopy water content we are solving for $d_w$


To calculate the absorption coefficients  $(\alpha(\lambda))$ , we use the [equation](../algorithms/EWT.py#L56-L81):

$$
\alpha(\lambda) = \frac{4\pi k(\lambda)}{\lambda}
$$


Where $k(\lambda)$ is the imaginary part of the refractive index of water at a given wavelength. The reflectance of the canopy $\rho(\lambda)$ is modeled as:
and to obtain $k(\lambda)$, we use the lab measurements [csv](../algorithms/k_liquid_water_ice.csv) from Kedenburg et al. (2012) for liquid water.


$$
\rho_{\text{model}}(\lambda) = (a + b \lambda) e^{-\alpha(\lambda) L}
$$

The residuals between the modeled reflectance and the measured reflectance are minimized using least squares optimization.

A least squares optimization is employed to solve for the path length L and the linear parameters 𝑎 (intercept) and b (slope). The optimization seeks to minimize the sum of squared residuals between the modeled and measured reflectance, iterating to find the optimal values for these parameters. This then gives us the liquid water path length based on the Beer-Lambert surface model, see the [python function](../algorithms/EWT.py#L87-L167)

#### 4.2.1 Mathematical theory assumptions
We assume that the absorption coefficents are constant.

### 4.3 Algorithm Input Variables
The required input files for EWT production are in Table 1.

**Table 1.** _Input variables_
| Name | Description | Unit | Required |
| --- | --- | --- | --- |
| HDRF (Surface Reflectance) | hemispherical-directional reflectance factor per wavelength | unitless | true |
| Endmember Spectral Library | collection of spectral surface endmembers (HDRF per wavelength) | unitless | true |
| Observation Geometry | solar zenith angle, view zenith angle, relative azimuth angle | degree | false |

### 4.4 Algorithm Output Variables
The SBG-VSWIR output data products delivered to the DAAC use their formatting conventions, the system operates internally on data products stored as binary data cubes with detatched human-readable ASCII header files.    


## 5 Algorithm Usage Constraints

## 6 Performance Assessment
There are currently no requirements to validate terrestrial vegetation products on a global basis; however, we will be conducting spot verifications for specific demonstration prodcuts. 

### 6.1 Validation Methods

### 6.2 Uncertainties

Bohn et al. (2020) observed a consistent overestimation in retrieved CWC, with values exceeding actual measurements by a factor of 2.5 to 3.5 (Figure 2). This discrepancy arises primarily from the application of the Beer-Lambert model, which does not consider multiple scattering effects within the canopy (Zhang et al., 2011). Additionally, beyond 0.5 g cm⁻², CWC estimates tend to stabilize due to absorption saturation. Despite these biases, the overall variability, significance, and interpretability of CWC maps remains unaffected. 
![Uncertainty CWT Calc](../fig/Uncertainty_ewt.PNG)

<figure>
    <img src="fig/Uncertainty_ewt.PNG" alt="Figure 3. Uncert EWT" width="500" height = "200">
    <figcaption style="font-style: italic; margin-bottom: 20px;">Figure 3: Comparison of retrieved canopy water content (CWC) with measured CWC from the ESA Barrax SPARC'03 field campaign. Error bars indicate the standard deviation of retrieved CWC (Bohn et al. 2020). </figcaption>
</figure> 


### 6.3 Validation Errors

## 7 Algorithm Implementation

### 7.1 Algorithm Availability
The SBG-VSWIR mission will store the specific implementation of vegetation model development and parameters, including configuration options, training datasets, etc. at the SBG software repository (https://github.com/nasa-sbg). 

### 7.2 Input Data Access
All input data required to run the code not found in the above repositories is available at the NASA Digital Archive. 

### 7.3 Output Data Access
All output data is available at the NASA Digital Archive and searchable from NASA Earthdata Search (https://search.earthdata.nasa.gov/search). 

### 7.4 Important Related URLs
N/A 

## 8 Significance Discussion

This ATBD describes the implementation of equivalent water thickness estimation.

## 9 Open Research
All code described here is open source.  All publications sponsored by the SBG-VSWIR mission related to this code are open access. 
## 10 Acknowledgements

## 11 Contact Details

K. Dana Chadwick 

ORCID: 0000-0002-5633-4865 

URL:  

Email: dana.chadwick@jpl.nasa.gov 

Role(s) related to this ATBD: writing - original and revision, methodology, software. 

Affiliation – Jet Propulsion Laboratory, California Institute of Technology 


C. Ade 

ORCID: 

Email: christiana.ade@jpl.nasa.gov

Role(s) related to this ATBD: writing - original and revision, methodology, software. 

Affiliation – Jet Propulsion Laboratory, California Institute of Technology 

## References
* Cheng, T., Rivard, B., Sánchez-Azofeifa, A., Feng, J., Calvo-Polanco, M., & Rencz, A. (2020). Monitoring drought-induced stress in vegetation using hyperspectral data. Remote Sensing of Environment, 237, 111466.

* Bohn, N., Guanter, L., Kuester, T., Preusker, R., & Segl, K. (2020). Coupled retrieval of the three phases of water from spaceborne imaging spectroscopy measurements. Remote Sensing of Environment, 242, 111708.

* Gao, B.-C. (1996). NDWI—A normalized difference water index for remote sensing of vegetation liquid water from space. Remote Sensing of Environment, 58(3), 257–266. 

* Green, R. O., Painter, T. H., Roberts, D. A., & Dozier, J. (2006). Measuring the expressed abundance of the three phases of water with an imaging spectrometer over melting snow. Water Resources Research, 42(10).

* Kedenburg, S., Vieweg, M., Gissibl, T., Giessen, H. (2012). Linear refractive index and absorption measurements of nonlinear optical liquids in the visible and near-infrared spectral region. Opt. Mater. Express 2, 1588–1611.

* Tucker, C. J. (1980). Remote sensing of leaf water content in the near infrared. Remote Sensing of Environment, 10(1), 23–32. 

* Ustin, S. L., Roberts, D. A., Pinzón, J., Jacquemoud, S., Gardner, M., Scheer, G., Castañeda, C. M., & Palacios-Orueta, A. (1998). Estimating canopy water content of chaparral shrubs using optical methods. Remote Sensing of Environment, 65(3), 280–291.