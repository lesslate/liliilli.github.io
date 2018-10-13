# PhysX sample `SampleBridges` 분석

> 2018-10-13 AM 01:08

월드의 구현은 `void SampleBridges::onInit()` 를 실행함으로써 환경이 구축됨.
그리고 도중의 `importRAWFile` 이라는 함수로 스택을 계속 쌓아감.

뭔가 `SampleBridges.raw` 라는 파일을 불러들이는 것 같음.

그리고 나서 쭈욱 계층이 올라가다가, 다시 본 `.cpp` 파일로 돌아와서, (`newShape`)
`static void buildBridge(PxScene& scene,...` 라는 함수를 호출한다. 여기서 진짜 월드를 만들어낸다!

``` c++
// create ground under character
if(1)
{
  PxTransform boxPose = tr * PxTransform(
      PxVec3(0.0f, -groundHeight, -10.0f-groundLength)*scale, PxQuat(PxIdentity)
  );
  boxPose.p += offset;

  const PxVec3 boxSize = PxVec3(1.0f, groundHeight, groundLength) * scale;
  PxRigidActor* box = 
      createBox(scene, physics, boxPose, boxSize, 0.0f, material, NULL, 0);
  actors.push_back(box);
}
```

위 코드는 주인공 스팟 지점의 그라운드를 만들어 낸다. 다만 실제 `API` 을 일일히 불러서 `PxShape` 등을 만들어내지는 않는다. 대신,

1.  `createBox => createRigidActor` 함수를 불러서 
2. `PxRigidActor` 와 `PxShape` 을 만들고 (`PxRigidActor` 는 `PxRigidDynamic` 의 윗형) 그런데 `physics.createRigidDynamic` 으로 이미 아랫형을 만들고 윗형으로 고쳤다가 다시 다운캐스팅.
3. `PxShape` 에 시뮬레이션 필터링을 거친 다음에
4. `Interia` `Mass` 등을 설정한 후에
5. `scene.addActor` 로 액터를 피직스에 설정.

``` c++
if(1) // ~298line
{
  const PxReal plankWidth = 1.0f;
  const PxReal plankHeight = 0.02f;
  const PxReal plankDepth = 0.5f;
  const PxReal plankDensity	= 0.03f;
  const PxReal plankDensity	= 0.3f;
  PxRigidDynamic* lastPlank	= 0;
  // ...
```

위 코드는 브릿지를 만든다. 

그리고 `299 ~ 307` 줄까지는 브릿지를 건넌 뒤의 그라운드를 만든다.

> 2018-10-13 AM 01:29

그 후에 다시 `SampleBridges::newShape(const RAWShape& data)` 로 돌아와서 만들어진 `PxActor` 들을 가지고 렌더 박스를 만들어낸다.
`createRenderBoxFromShape(bridgeActors[i], boxShape, m, gBoxUVs);`

> 2018-10-13 AM 01:39

`createShape` 의 플래그를 끄고 어느것이 생성이 되지 않는지를 봤는데, 맨 위의 박스 생성은 캐릭터 밑의 발판이 아니라 브릿지에 연결할 기둥의 하나였음. 

반대로 맨 밑의 박스 생성이 캐릭터 밑의 발판이었고... 하... 브릿지는 기둥이 없어도 고정대가 생성되서 그걸 축으로 해서 브릿지를 만드나봄.

> 2018-10-13 AM 01:44

그리고 `SampleBridges::newShape` 에서 `/else if(::strcmp(data.mName, "Platform")==0)` 밑으로 되있는 것은, 플랫폼을 생성하기 위한 것임. 

움직이는 플랫폼을 생성할 때도 `PxRigidDynamic` 을 생성하는 것은 같음.
다만

``` c++
#ifdef CCT_ON_BRIDGES
platformActor->setName(gPlatformName);
#endif
platformActor->setRigidBodyFlag(PxRigidBodyFlag::eKINEMATIC, true);
```

로 키네마틱으로 설정해서 움직도록 함.

또한 `KinematicPlatform` 이라는 별도의 형을 생성해서, 거기에 만들어진 플랫폼의 `PxRigidDynamic` 에 대한 포인터를 가지도록 하고 있음. 뿐만 아니라 시간, 위치, 스피드 및 애니메이션 모드 (`mMode`) 의 정보를 갱신하고 있음. 

그리고 여기서는 왜 이따구로 했는지 이해가 안간다만... 해당 플랫폼의 경유지, 목적지 역시 갱신하고 있음.

``` c++
void KinematicPlatform::init(PxU32 nbPts, const PxVec3* pts, const PxTransform& globalPose, const PxQuat& localRot, PxRigidDynamic* actor, PxReal travelTime, PxReal rotationSpeed, LoopMode mode)
{
	DELETEARRAY(mPts);
	mNbPts	= nbPts;
	mPts	= SAMPLE_NEW(PxVec3Alloc)[nbPts];
	PxVec3* dst = mPts;
	for(PxU32 i=0;i<nbPts;i++)
		dst[i] = globalPose.transform(pts[i]);

	mLocalRot		= localRot;
	mActor			= actor;
	mTravelTime		= travelTime;
	mRotationSpeed	= rotationSpeed;
	mMode			= mode;
	PX_ASSERT(travelTime>0.0f);
}
```

## 다른 사이트에서

**OnInit()** 

-  importRAWFile("SampleBridges.raw", gGlobalScale); 
-  importRAWFile("sky_mission_race1.raw", 1.0f); 
-  controllActor 초기화 
-  mCCTCamera 카메라 초기화 


 **ControlledActor** 

-  PxBoxController 를 생성하고 PxControllerManager 에 등록한다. 
-  mRenderActorStanding, mRenderActorCrouching 생성해서 관리한다. 
-  ControlledActor 생성  
-  출력될 RenderbaseActor 등록. 

**SampleCCTCameraController** 

-  CameraController 상속 
-  키입력을 받아서 ControlActor 를 이동시킨다. 

> 2018-10-13 AM 02:29

`ControlledActor` 는 Sample 에서 직접 만든 것.

> https://docs.nvidia.com/gameworks/content/gameworkslibrary/physx/guide/Manual/CharacterControllers.html

``` c++
void PhysXSample::onTickPreRender(float dtime)
{
	// ...
    PxSceneWriteLock scopedLock(*mScene);
    mApplication.baseTickPreRender(dtime);
    // ...
}

void PhysXSampleApplication::baseTickPreRender(float dtime)
{
	if(mCurrentCameraController && !isConsoleActive())
		mCurrentCameraController->update(getCamera(), dtime);
}

void SampleCCTCameraController::update(Camera& camera, PxReal dtime)

// ...
    
const PxU32 flags = mCCTs[i]->mController->move(disp, 0.0f, dtime, filters, mObstacleContext);
```

로 컨트롤러가 달린 (키 입력 등에 따라 움직일 수 있는) 액터들을 움직이게 한다. (`PhysX` 에게 보낸다는 것인듯)

그리고 플랫폼의 경우

``` c++
void SampleBridges::onSubstep(float dtime)
{
	mPlatformManager.updatePhysicsPlatforms(dtime);
}

void KinematicPlatformManager::updatePhysicsPlatforms(float dtime)
{
	// PT: keep track of time from the point of view of physics platforms.
	// - if we call this each substep using fixed timesteps, it is never exactly in sync with the render time.
	// - if we drop substeps because of the "well of despair", it can seriously lag behind the render time.
	mElapsedPlatformTime += dtime;

	// PT: compute new positions for (physics) platforms, then 'setKinematicTarget' their physics actors to these positions.
	const size_t nbPlatforms = mPlatforms.size();
	for(PxU32 i=0;i<nbPlatforms;i++)
		mPlatforms[i]->updatePhysics(dtime);
}

// KinematicPlatform
PX_FORCE_INLINE	void updatePhysics(PxReal dtime)
{ updateState(mPhysicsState, NULL, dtime, true);
 
// FInally!
void KinematicPlatform::updateState(PlatformState& state, PxObstacleContext* obstacleContext, PxReal dtime, bool updateActor) const
{
	state.mCurrentTime += dtime;
	state.mCurrentRotationTime += dtime;

	// Comput
    // ...
}
```

에서 델타 타임을 가지고 업데이트를 한다. `dt` 는 0.0167초.

``` c++
PxVec3 currentPos;
if(getPoint(currentPos, state.mFlip ? 1.0f - t : t))
{
	const PxVec3 wp = currentPos;

	PxMat33 rotY;
	PxToolkit::setRotX(rotY, state.mCurrentRotationTime*mRotationSpeed);
	const PxQuat rotation(rotY);

	const PxTransform tr(wp, mLocalRot * rotation);
	state.mPrevPose = tr;
    // ...
```

계산한 값을 가지고 현재 인스턴스의 커스텀 구조체인 `PlatformState` 에 넣고 있다. 

``` c++
	if(updateActor)
	{
		PxSceneWriteLock scopedLock(*mActor->getScene());
		mActor->setKinematicTarget(tr);
    }
```

> https://docs.nvidia.com/gameworks/content/gameworkslibrary/physx/apireference/files/classPxRigidDynamic.html#4464d188e7a1e94582c9cf35da9bbc93

`Kinematic` 한 플랫폼 혹은 `PxRigidDynamic` 을 움직이게 한다. 매 프레임마다 호출되기 때문에, 플랫폼이 움직인다!

