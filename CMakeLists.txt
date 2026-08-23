set(FLUFFYTW
	${CMAKE_CURRENT_SOURCE_DIR}/f_helper.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/f_helper.h
	${CMAKE_CURRENT_SOURCE_DIR}/f_component.h
	${CMAKE_CURRENT_SOURCE_DIR}/f_config.h

	${CMAKE_CURRENT_SOURCE_DIR}/f_bots.h
	${CMAKE_CURRENT_SOURCE_DIR}/f_bots.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/aimbot/aimbot.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/aimbot/aimbot_scans.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/aimbot/aimbot.h

	${CMAKE_CURRENT_SOURCE_DIR}/f_visuals.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/f_visuals-base.cpp
	${CMAKE_CURRENT_SOURCE_DIR}/f_visuals.h
)

<<<<<<< HEAD
set(GAME_CLIENT
	${GAME_CLIENT}
	${FLUFFYTW}
	PARENT_SCOPE
)
=======
# 19.9 grabs client sources via GLOB_RECURSE, so just make sure ours end up in GAME_CLIENT
foreach(f ${FLUFFYTW})
	if(NOT f IN_LIST GAME_CLIENT)
		list(APPEND GAME_CLIENT ${f})
	endif()
endforeach()
set(GAME_CLIENT ${GAME_CLIENT} PARENT_SCOPE)
>>>>>>> bc978f2 (19.9 port)
