---
layout: single
title: "Bilinear interpolation"
tags: 
date: 2018-06-26 +0900
categories: Graphics
comments: false
---
<script type="text/javascript"
    src="http://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML">
</script>

### Bilinear interpolation 이란?

### 전체 코드

{% highlight glsl %}
// Bilinear interpolation by liliilli. (twitter: @NxNeu_J)
// 2018 Jun liliilli all rights reserved.

#define MACRO_BLACK vec3(0)
#define MACRO_WHITE vec3(1)
#define MACRO_THRE_BLACK vec3(0.45)
#define MACRO_THRE_WHITE vec3(0.55)

vec3 checkboard[] = vec3[4](
    MACRO_BLACK, MACRO_WHITE,
    MACRO_WHITE, MACRO_BLACK
);

void mainImage( out vec4 fragColor, in vec2 fragCoord ) {
    vec2 puv = mod(fragCoord, 64.0) / 32.0;
    vec2 uv = puv - floor(puv);
    
    ivec2 xlyb = ivec2(floor(puv));
    ivec2 xryt = ivec2(0);
    if (xlyb.x != 1) { xryt.x = xlyb.x + 1; }
    if (xlyb.y != 1) { xryt.y = xlyb.y + 1; }
    
    ivec2 xryb = ivec2(xryt.x, xlyb.y);
    ivec2 xlyt = ivec2(xlyb.x, xryt.y);
    
    vec3 color =
    	(1.0-uv.x) * (1.0-uv.y) * checkboard[xlyb.y * 2 + xlyb.x] +
    	uv.x * (1.0-uv.y) * checkboard[xryb.y * 2 + xryb.x] +
    	(1.0-uv.x) * uv.y * checkboard[xlyt.y * 2 + xlyt.x] +
    	uv.x * uv.y * checkboard[xryt.y * 2 + xryt.x];
    color = smoothstep(MACRO_THRE_BLACK, MACRO_THRE_WHITE, color);
    
    fragColor = vec4(color.xyz, 1.0);
}
{% endhighlight %}

### 결과

![result]({{ "/assets/201806/jun23_screen_door_transparency.gif" | absolute_url }})

체크무늬로 반투명 처리가 됨을 확인할 수 있다.