Numerical integration is the process of approximating the value of a definite integral using numerical techniques such as summing function values at discrete points, rather than solving it analytically.
- Used when analytical integration is impractical or too computationally expensive
- Quadrature Integration and Stochastic Integration are common methods

## Quadrature Integration
Quadrature integration is the numerical approximation of definite integrals by computing weighted sums of function values at specific points.
$$
I = \int_{a}^{b} f(x) \, dx \approx \sum_{i=1}^{N} w_{i} f(x_{i})
$$
- xi are the evaluation points (also called nodes)
- wi are the corresponding weights
- N is the number of evaluation points
This approximation's accuracy depends on the choice of evaluation points and weights. Increasing the number of evaluation points improves accuracy and allows the estimate to converge to the exact integral. Methods for this approximation range from simple (Reimann sum) to advanced (Gaussian quadrature).

### Reimann Sum (Midpoint) Integration
![](../Images/Pasted%20image%2020261004144138.png)

The Riemann sum (or rectangle rule) is a numerical integration method that approximates the integral by summing rectangular areas under the curve. 
$$I \approx \sum_{i=1}^{N} f(x_{i}) \Delta x$$
- xi are the sample points chosen for subintervals
- Delta x is the interval width, calculated as (b-a)/N
Common variations include the left, right, and midpoint Reimann sums which use those spots on the intervals respectively. Note that this method is easy to implement but not as accurate as advanced methods.

### Trapezoidal Rule Integration
![](../Images/Pasted%20image%2020261004144605.png)
This is a slightly more accurate version using linear interpolation between sample points. The trapezoidal rule generally improves upon the rectangle method and can be calculated using the following equation: 
$$I \approx \sum_{i=1}^{N-1} \frac{f(x_{i}) + f(x_{i+1})}{2} \Delta x$$
- Delta x is the step size, calculated as (b-a)/(N-1)
- The function is evaluated at N evenly spaced points

## Higher Order Quadrature Techniques
These techniques provide more accurate approximations by better capturing function behavior within each interval. These methods may employ sophisticated weighting schemes and strategically chosen evaluation points for greater precision. Examples include Simpson's rule and Gaussian quadrature. Frequently used in applications that demand high precision, and these techniques have increased computational complexity.
## Stochastic Integration Methods
These methods approximate integrals by randomly sampling function values over the domain and using statistical techniques to estimate the result. This is suitable for high dimensional and complex functions, and they converge probabilistically.

### Monte Carlo Integration
![](../Images/Pasted%20image%2020261004145435.png)
$$I \approx \frac{b-a}{N} \sum_{i=1}^{N} f(x_{i})$$
- xi are random samples drawn uniformly from the range a to b
- The estimate converges as O(1/sqrt(N)) so it is relatively slow
This method estimates the integral by randomly sampling points within the domain and averaging the function values. It works well in high dimensions and is an unbiased estimator. 
The Monte Carlo method fails to accurately estimate the integral for a function with a single small important region over a wider range, since it uses the same weight for every point. The following image displays a function where Monte Carlo would not be a good fit: 
![](../Images/Pasted%20image%2020261004150042.png)
Importance sampling can be used to focus on the regions with higher importance. 

### Monte Carlo with Importance Sampling
$$I \approx \frac{1}{N} \sum_{i=1}^{N} \frac{f(x_{i})}{p(x_{i})}$$
- p(x) is a carefully chosen probability density function (PDF)
- The PDF reduces variance by focusing samples where f(x) is likely to be large
- The PDF controls both the selection of sample points and their corresponding weights in the estimation
- Choosing an appropriate p(x) is critical to achieving low-variance results

## The Reflectance Equation
The Reflectance Equation models the light reflected from a surface point by integrating incoming radiance over the hemisphere:
$$L_{o}(\mathbf{x}, k_{o}) = \int_{k_{i} \in \Omega} \rho(k_{i}, k_{o}) L_{i}(\mathbf{x}, k_{i}) \cos\theta_{i} \, dk_{i}$$
- p(ki, ko) is the BRDF Bidirectional Reflectance Distribution Function describing surface reflectance properties
- Li(x, ki) is the incoming radiance, representing the light arriving from direction ki
- cos(theta i) is Lambert's cosine law, weighting incoming light by its incident angle
- dki is a differential solid angle, integrating over all directions in the hemisphere     
The reflectance equation inherently requires recurse ray casting since incoming radiance depends on other surfaces. Recursive evaluation leads to computational challenges including exponential grown in ray interactions.  

## Naive Recursive Ray Tracing
```
getRadiance(ray):
	cast ray, hit point x, direction k
	if x is light source:
		return lightRadiance(x,k)
	radiance = 0
	for each direction on hemisphere around x:
		create reflRay in direction k'
		radiance += weightRadiance(p(x,k,k'), getRadiance(reflRay))
	return radiance
```
- p in this case is the BRDF
This process however requires sampling across all incident directions, which causes exponential computational growth.
![](../Images/Pasted%20image%2020261004152824.png)

## The Rendering Equation (Kajiya's)
This equation provides a unified framework for light transport by considering:
- Direct and indirect illumination
- Recursive evaluation of light ineractions
- Integration over all incoming light directions
$$L_{o}(k_{o}) = L_{e}(k_{o}) + \int_{k_{i} \in \Omega} \rho(k_{i}, k_{o}) L_{i}(k_{i}) \cos\theta_{i} \, dk_{i}$$
- Lo(ko) is the outgoing radiance
- Le is the emitted radiance
- p(ki,ko) is the BRDF
- Li(ki) is the incoming radiance
- cos(theta i) is Lambert's cosine law
- dki is the differential solid angle
Kajiya recognized that Monte Carlo integral estimation with importance sampling is necessary to manage computational load and focus measurements on significant contributions. We can sample towards known visible light sources to capture direct illumination efficiently, rather than importance sampling dark areas that wouldn't contribute as much to the scene. Also, sample directions based on the BRDF to prioritize significant reflective contributions.
## Path Tracing
Path tracing is the practical application of Monte Carlo integration to solve the rendering equation by:
- Recursively tracing light paths from the camera
- Samples a single path per iteration through the scene, rather than recursively integrating at all contributing surfaces
- Accumulates contributions from multiple bounces 
While this approach is accurate, it requires many samples to converge. 

## Path Tracing Pseudocode
```
get_radiance(ray):
    cast ray into scene, hit at x,
        normal N, outgoing direction k

    radiance = emit_radiance(x, k)

    foreach light in scene:
        if sees_light(x, light):
            light_dir = direction_to(light, x)
            radiance +=
                weight_radiance(ρ(x, k, light_dir), get_radiance(light_dir))

    if random < schlick_fresnel(x, k, N):
        refl_ray, k' = importance_sample(x, k, ρ)
        radiance += weight_radiance(ρ(x, k, k'), get_radiance(refl_ray))
    else:
        trans_ray, k' = importance_sample(x, k, τ)
        radiance += weight_radiance(τ(x, k, k'), get_radiance(trans_ray))

    return radiance
```
