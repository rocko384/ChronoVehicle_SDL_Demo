SET(BASE_FILES
	demo_HMMWV_SDL.cpp
)

SET(DRIVER_FILES
	driver/ChIrrGuiDriver_SDL.cpp
	driver/ChIrrGuiDriver_SDL.h
)

SET(INPUT_FILES
	input/SDLInputManager.cpp
	input/SDLInputManager.h
)

SET(MODEL_FILES
	models/ModelDefs.h
	models/hmmwv/HMMWV_FuncDriver.h
	models/hmmwv/HMMWV_FuncDriver.cpp
	models/hmmwv/vehicle/HMMWV_Vehicle.h
	models/hmmwv/vehicle/HMMWV_Vehicle.cpp
	models/hmmwv/vehicle/HMMWV_VehicleJSON.h
	models/hmmwv/vehicle/HMMWV_VehicleJSON.cpp
	models/hmmwv/suspension/HMMWV_DoubleWishbone.h
	models/hmmwv/suspension/HMMWV_DoubleWishbone.cpp
	models/hmmwv/steering/HMMWV_PitmanArm.h
	models/hmmwv/steering/HMMWV_PitmanArm.cpp
	models/hmmwv/driveline/HMMWV_Driveline2WD.h
	models/hmmwv/driveline/HMMWV_Driveline2WD.cpp
	models/hmmwv/driveline/HMMWV_Driveline4WD.h
	models/hmmwv/driveline/HMMWV_Driveline4WD.cpp
	models/hmmwv/powertrain/HMMWV_SimplePowertrain.h
	models/hmmwv/powertrain/HMMWV_SimplePowertrain.cpp
	models/hmmwv/powertrain/HMMWV_Powertrain.h
	models/hmmwv/powertrain/HMMWV_Powertrain.cpp
	models/hmmwv/brake/HMMWV_BrakeSimple.h
	models/hmmwv/brake/HMMWV_BrakeSimple.cpp
	models/hmmwv/wheel/HMMWV_Wheel.h
	models/hmmwv/wheel/HMMWV_Wheel.cpp
	models/hmmwv/tire/HMMWV_RigidTire.h
	models/hmmwv/tire/HMMWV_RigidTire.cpp
	models/hmmwv/tire/HMMWV_LugreTire.h
	models/hmmwv/tire/HMMWV_LugreTire.cpp
)

SOURCE_GROUP("driver" FILES ${DRIVER_FILES})
SOURCE_GROUP("input" FILES ${INPUT_FILES})
SOURCE_GROUP("models" FILES ${MODEL_FILES})
SOURCE_GROUP("" FILES ${BASE_FILES})


SET(LIBRARIES 
	${CHRONOENGINE_LIBRARIES}
	${CHRONOVEHICLE_LIBRARIES}
	${CHRONOVEHICLE_IRRLICHT_LIB}
	${CHRONOVEHICLE_UTILS_LIB}
	${CH_SDL2LIB}
	${CH_SDL2mainLIB}
)

ADD_EXECUTABLE(ChronoVehicle_SDL_Demo ${BASE_FILES} ${DRIVER_FILES} ${INPUT_FILES} ${MODEL_FILES})
SET_TARGET_PROPERTIES(ChronoVehicle_SDL_Demo PROPERTIES 
                      COMPILE_FLAGS "${CH_BUILDFLAGS}"
                      LINK_FLAGS "${LINKERFLAG_EXE}")
TARGET_LINK_LIBRARIES(ChronoVehicle_SDL_Demo ${LIBRARIES})
INSTALL(TARGETS ChronoVehicle_SDL_Demo DESTINATION bin)