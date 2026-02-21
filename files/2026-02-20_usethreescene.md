# useThreeScene

**Type:** File Overview
**Repository:** angelamos-3d
**File:** frontend/src/hooks/useThreeScene.ts
**Language:** typescript
**Lines:** 1-248
**Complexity:** 0.0

---

## Source Code

```typescript
// ===================
// © AngelaMos | 2026
// useThreeScene.ts
// ===================

import { type VRM, VRMLoaderPlugin } from "@pixiv/three-vrm";
import { useEffect, useRef } from "react";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { getAngelaConfig } from "../config";
import { AnimationController, AnimationManager } from "../lib/animation";
import { logger } from "../lib/debug";

interface UseThreeSceneResult {
	canvasRef: React.RefObject<HTMLCanvasElement | null>;
	vrmRef: React.MutableRefObject<VRM | null>;
	animControllerRef: React.MutableRefObject<AnimationController | null>;
	animManagerRef: React.MutableRefObject<AnimationManager | null>;
	analyserRef: React.MutableRefObject<AnalyserNode | null>;
	dataArrayRef: React.MutableRefObject<Uint8Array<ArrayBuffer> | null>;
	isPlayingRef: React.MutableRefObject<boolean>;
}

export function useThreeScene(): UseThreeSceneResult {
	const canvasRef = useRef<HTMLCanvasElement | null>(null);
	const vrmRef = useRef<VRM | null>(null);
	const animControllerRef = useRef<AnimationController | null>(null);
	const animManagerRef = useRef<AnimationManager | null>(null);
	const analyserRef = useRef<AnalyserNode | null>(null);
	const dataArrayRef = useRef<Uint8Array<ArrayBuffer> | null>(null);
	const isPlayingRef = useRef(false);

	useEffect(() => {
		const canvas = canvasRef.current;
		if (!canvas) return;

		const config = getAngelaConfig();
		const width = window.innerWidth;
		const height = window.innerHeight;

		const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
		renderer.setPixelRatio(window.devicePixelRatio);
		renderer.setSize(width, height);
		renderer.outputColorSpace = THREE.SRGBColorSpace;

		const scene = new THREE.Scene();
		const bgColor = new THREE.Color(config.scene.backgroundColor);
		scene.background = bgColor;
		scene.fog = new THREE.FogExp2(bgColor.getHex(), 0.1);

		const grid = new THREE.GridHelper(200, 200, 0xffffff, 0xffffff);
		grid.position.y = 0;
		grid.material.opacity = 0.15;
		grid.material.transparent = true;
		scene.add(grid);

		const starCount = 50;
		const starPositions = new Float32Array(starCount * 3);
		const starPhases = new Float32Array(starCount);
		const starSpeeds = new Float32Array(starCount);

		for (let i = 0; i < starCount; i++) {
			starPositions[i * 3] = (Math.random() - 0.5) * 30;
			starPositions[i * 3 + 1] = Math.random() * 15 + 2;
			starPositions[i * 3 + 2] = -Math.random() * 20 - 5;
			starPhases[i] = Math.random() * Math.PI * 2;
			starSpeeds[i] = 0.3 + Math.random() * 0.4;
		}

		const starGeometry = new THREE.BufferGeometry();
		starGeometry.setAttribute(
			"position",
			new THREE.BufferAttribute(starPositions, 3),
		);

		const starMaterial = new THREE.ShaderMaterial({
			uniforms: {
				uTime: { value: 0 },
			},
			vertexShader: `
        attribute float phase;
        attribute flo
```

---

## File Overview

# useThreeScene.ts Documentation

## Purpose and Responsibility
This file is responsible for initializing and managing a 3D scene within a React application using Three.js. It sets up the canvas, renderer, camera, lighting, and VRM model loading. The scene includes stars, grid helpers, and ambient/directional lights to create an immersive environment.

## Key Exports or Public Interface
- `UseThreeSceneResult`: An interface defining the result of the `useThreeScene` hook, containing references to various components like canvas, VRM model, animation controllers, analyser node, and more.
- `useThreeScene()`: A React custom hook that initializes and manages the 3D scene setup.

## How It Fits in the Project
This file is a crucial part of the frontend application, handling the rendering logic for the 3D scenes. It integrates with other components and services defined in the project, such as animation controllers from `../lib/animation` and configuration settings from `../config`. The hook is likely used within higher-order components or specific scene-rendering contexts to provide a consistent 3D environment.

## Notable Design Decisions
- **Custom Hook**: Utilizes React's custom hook pattern to encapsulate complex state management and side effects.
- **Three.js Setup**: Configures the Three.js renderer, camera, lighting, and VRM model loading with detailed settings for realism and performance.
- **Animation Management**: Integrates `AnimationController` and `AnimationManager` from a local library to handle smooth animations of the VRM model.
- **Star Effect**: Implements a custom star effect using shaders to create a dynamic background.
- **Dependency Injection**: Uses `useRef` to manage mutable state across render cycles, ensuring consistent access to scene components.

---

*Generated by CodeWorm on 2026-02-20 23:42*
