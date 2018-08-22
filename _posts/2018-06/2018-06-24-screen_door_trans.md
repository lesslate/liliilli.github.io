---
layout: single
title: "Screen-door transparency"
tags: 
date: 2018-06-24 +0900
categories: Graphics
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

### Screen-door Transparency technique 란?

### 전체 코드

{% highlight glsl %}
precision highp float;
uniform float time;
uniform vec2 resolution;
varying vec3 fPosition;
varying vec3 fNormal;

#define M_PI 3.1415926535
#define M_PI8 (M_PI * 8.0)

mat4 threshold_matrix = mat4(
1.0 / 17.0,  9.0 / 17.0,  3.0 / 17.0, 11.0 / 17.0,
13.0 / 17.0,  5.0 / 17.0, 15.0 / 17.0,  7.0 / 17.0,
4.0 / 17.0, 12.0 / 17.0,  2.0 / 17.0, 10.0 / 17.0,
16.0 / 17.0,  8.0 / 17.0, 14.0 / 17.0,  6.0 / 17.0
);

// View space...
float light_intensity = 1.5;
vec3 light_irradiance = vec3(1, 1, 1) * light_intensity;
vec3 light_vector = -normalize(vec3(0, -1, -1));
vec3 model_diffuse = vec3(1, 1, 1);
vec3 model_specular = vec3(1, 1, 1);
float model_smoothness = 10.0;

struct DLight {
  float light_intensity;
  vec3 light_diffuse;
  vec3 light_vector;
} lights[1];

void init() {
  lights[0] = DLight(1.0, vec3(1, 1, 1) , -normalize(vec3(0, -1, -0.25)));
}

vec3 get_radiance_color() {
  vec3 view_vector = normalize(-fPosition);
  vec3 k_s = (model_smoothness + 8.0) / M_PI8 * model_specular;
  vec3 k_d = model_diffuse / M_PI;
  
  vec3 color = vec3(0);
  for (int i = 0; i < 1; i++) {
    vec3 half_vector = normalize(lights[i].light_vector + view_vector);
    float cos_half_spec = max(0.0, dot(half_vector, fNormal));
    float cos_norm_diff = max(0.0, dot(lights[i].light_vector, fNormal));

    color += (k_d + k_s * cos_half_spec) * cos_norm_diff * 
        (lights[i].light_diffuse  * lights[i].light_intensity);
  }
  
  return color;
}

void process_screen_door_transparency_discard(float alpha) {
  int x = int(mod(gl_FragCoord.x, 4.0));
  int y = int(mod(gl_FragCoord.y, 4.0));
  
  if (x == 0) {
    if (y == 0) {
      if (alpha - threshold_matrix[0][0] < 0.0) { discard; }
    }
    else if (y == 1) {
      if (alpha - threshold_matrix[1][0] < 0.0) { discard; }
    }
    else if (y == 2) {
      if (alpha - threshold_matrix[2][0] < 0.0) { discard; }
    }
    else {
      if (alpha - threshold_matrix[3][0] < 0.0) { discard; }
    }
  }
  else if (x == 1) {
    if (y == 0) {
      if (alpha - threshold_matrix[0][1] < 0.0) { discard; }
    }
    else if (y == 1) {
      if (alpha - threshold_matrix[1][1] < 0.0) { discard; }
    }
    else if (y == 2) {
      if (alpha - threshold_matrix[2][1] < 0.0) { discard; }
    }
    else {
      if (alpha - threshold_matrix[3][1] < 0.0) { discard; }
    }
  }
  else if (x == 2) {
    if (y == 0) {
      if (alpha - threshold_matrix[0][2] < 0.0) { discard; }
    }
    else if (y == 1) {
      if (alpha - threshold_matrix[1][2] < 0.0) { discard; }
    }
    else if (y == 2) {
      if (alpha - threshold_matrix[2][2] < 0.0) { discard; }
    }
    else {
      if (alpha - threshold_matrix[3][2] < 0.0) { discard; }
    }
  }
  else {
    if (y == 0) {
      if (alpha - threshold_matrix[0][3] < 0.0) { discard; }
    }
    else if (y == 1) {
      if (alpha - threshold_matrix[1][3] < 0.0) { discard; }
    }
    else if (y == 2) {
      if (alpha - threshold_matrix[2][3] < 0.0) { discard; }
    }
    else {
      if (alpha - threshold_matrix[3][3] < 0.0) { discard; }
    }
  }
}

void main()
{
  init();
  
  float alpha = (sin(time * 40.0) + 1.0) / 2.0;
  process_screen_door_transparency_discard(alpha);
  
  vec3 color = get_radiance_color();
  gl_FragColor = vec4(color, 1.0);
}
{% endhighlight %}

### 결과

![result]({{ "/assets/201806/jun23_screen_door_transparency.gif" | absolute_url }})

체크무늬로 반투명 처리가 됨을 확인할 수 있다.