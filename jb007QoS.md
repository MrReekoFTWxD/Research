
> James Bond 007 QoS 

| Rendering Functions      | Offsets | Args |
| ----------- | ----------- | ---|
| R_AddCmdDrawPic | 0x821AEDC0 | float x, float y, float w, float h, float s0, float t0, float s1, float t1, const float* color, Material* material | 
| Material_RegisterHandle | 0x82565398 | const char* name, int img |
| R_RegisterFont | 0x821BD4F0 | const char* name, int img |
| R_AddCmdDrawText | 0x82572438 | const char *text, int maxChars, Font *font, float x, float y, float xScale, float yScale, float rotation, const float *color, int style |
| R_TextWidth | 0x82564C58 | const char* text, int maxchar, Font* font |
| R_TextHeight | 0x82564D48 | Font *font |
> Rendering Structs
```cpp
struct Material
{
    const char* name;
}
struct Font
{
	const char* name;
	int pixelHeight;
};
```


| Random Functions      | Offsets | Args |
| ----------- | ----------- | ---|
| Cbuf_AddText | 0x82298750 | int local, char* cmd |
| SL_GetString | 0x822AF3B0 | const char* str, int localController |
| AimTarget_GetTagPos | 0x821261D0 | int local, centity_t const* cen, unsigned int, vec3_t &out |
| CG_CompassDrawPlayerMap | 0x8216D060 | int localClientNum, CompassType compassType, rectDef_s *parentRect, rectDef_s *rect, Material *material, const float *color |
| CG_CompassDrawPlayer | 0x8216D568 | int localClientNum, CompassType compassType, rectDef_s *parentRect, rectDef_s *rect, Material *material, const float *color |
> Compass Structs
```cpp
enum CompassType
{
	COMPASS_TYPE_PARTIAL,
	COMPASS_TYPE_FULL, 
};

struct rectDef_s
{
	float x, y, w, h;
	int horzAlign, vertAlign;
};
```

| Hook      | Offsets | Args |
| ----------- | ----------- | ---|
| Menu_PaintAll | 0x822D90C8 | UiContext* dc, menuDef* menu | 

| Pointer      | Offsets | Size |
| ----------- | ----------- | ---|
| UiContext | 0x83725CD0 | 0x6E5C |
| ScrPlace | 0x828A1708 | 0xBC | 
| cg_entityArray | 0x82673F80 | 0x1EC |
| cg_t | 0x82770180 | FUCKING HUGE |

>Pointer Structs
```cpp
enum UIContextIndex_t : DWORD
{
  INVALID_UI_CONTEXT = 0xFFFFFFFF,
  UI_CONTEXT_FRONTEND = 0x0,
  UI_CONTEXT_INDEX_0 = 0x0,
  UI_CONTEXT_COUNT = 0x1,
};

struct UiContext
{
	UIContextIndex_t contextIndex; //0x0000
	float bias; //0x0004
	int realTime; //0x0008
	int frameTime; //0x000C
	float cursorX; //0x0010
	float cursorY; //0x0014
	int isCursorVisible; //0x0018
	int screenWidth; //0x001C
	int screenHeight; //0x0020
	float screenAspect; //0x0024
	float FPS; //0x0028
	float blurRadiusOut; //0x002C

};

struct refdef_t
{
	int x; //0x0000
	int y; //0x0004
	int width; //0x0008
	int height; //0x000C
	float tanHalfFovX; //0x0010
	float tanHalfFovY; //0x0014
	char _0x18[0x30];
	vec3_t eyePos; //0x0048
	vec3_t eyePosOld; //0x0054
	vec3_t viewAxis[3]; //0x0060
};

struct cg_t
{
  char _0x00[0x1E34C];
  refdef_t refdef;
};

enum entityTypes {
	ET_GENERAL,
	ET_PLAYER,
	ET_PLAYER_SHADOW,
	ET_ITEM,
	ET_MISSILE,
	ET_INVISIBLE,
	ET_SCRIPTMOVER,
	ET_SOUND_BLEND,
	ET_FX,
	ET_LOOP_FX,
	ET_PRIMARY_LIGHT,
	ET_MG42,
	ET_ACTOR,
	ET_ACTOR_SPAWNER,
	ET_ACTOR_CORPSE,
	ET_VEHICLE, //enemys are cg_entities[i].eType = 0xF ??? idk why maybe using wrong etype but worked for me
	ET_VEHICLE_CORPSE,
	ET_VEHICLE_COLLMAP,
	ET_INTERACTABLE,
	ET_DOOR,
	ET_ZONE_EMITTER,
	ET_TRIGGER,
};

struct centity_t
{
	char pad_0000[2]; //0x0000
	unsigned __int8 eType; //0x0002
	char pad_0003[5]; //0x0003
	vec3_t origin; //0x0008
	vec2_t viewAngles; //0x0014
	char pad_001C[464]; //0x001C
}; //Size: 0x01EC
```
> How I filled the structs
```cpp
UiContext* cgDC = (UiContext*)0x83725CD0;
cg_t* cg = (cg_t*)0x82770180;
centity_t* cg_entitiesArray = (centity_t*)0x82673F80;
```

> WorldToScreen
```cpp
float DotProduct(vec3_t vec1, vec3_t vec2) {
	return (vec1.x * vec2.x) + (vec1.y * vec2.y) + (vec1.z * vec2.z);
}

bool WorldToScreen(vec3_t worldPos, vec2_t* screenPos)
{
	vec3_t Pos = worldPos - cg->eyePos;
	vec3_t Transform;

	Transform.x = DotProduct(Pos, cg->viewAxis[1]);
	Transform.y = DotProduct(Pos, cg->viewAxis[2]);
	Transform.z = DotProduct(Pos, cg->viewAxis[0]);

	if (Transform.z < 0.1f)
		return false;
	vec2_t Center = vec2_t(cg->width / 2, cg->height / 2);
	screenPos->x = Center.x * (1 - (Transform.x / cg->fovX / Transform.z));
	screenPos->y = Center.y * (1 - (Transform.y / cg->foxY / Transform.z));
}
```

>Bone Strings 
```cpp

const char *boneStr[] = {
	"left_hand_ik",
	"right_hand_ik",
	"tag_flash",
	"tag_flash_s",
	"drone",
	"tag_origin",
	"mainroot",
	"spinelower",
	"spinemid",
	"spineupper",
	"neck",
	"head",
	"l_clavicle",
	"l_upperarm",
	"l_forearm",
	"l_wrist",
	"l_wrist_twist",
	"r_clavicle",
	"r_upperarm",
	"r_forearm",
	"r_wrist",
	"r_wrist_twist",
	"l_thigh",
	"l__calf",
	"l_foot",
	"r_thigh",
	"r_calf",
	"r_foot",
	"tag_weapon_right",
	"j_ring_le_1",
	"l_index_1",
	"l_mid_1",
	"l_pinky_1",
	"l_thumb_1",
	"l_ring_2",
	"l_index_2",
	"l_mid_2",
	"l_pinky_2",
	"l_thumb_2",
	"l_ring_3",
	"l_index_3",
	"l_mid_3",
	"l_pinky_3",
	"l_thumb_3",
	"r_ring_1",
	"r_index_1",
	"r_mid_1",
	"r_pinky_1",
	"r_thumb_1",
	"r_ring_2",
	"r_index_2",
	"r_mid_2",
	"r_pinky_2",
	"r_thumb_2",
	"r_ring_3",
	"r_index_3",
	"r_mid_3",
	"r_pinky_3",
	"r_thumb_3"
};
```
