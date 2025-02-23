project(stk CXX)


option(REALTIME "Realtime support" ON)
option(ENABLE_JACK "Enable JACK" ON)
option(ENABLE_ALSA "Enable ALSA API support (linux only)" ON)
set(CMAKE_C_FLAGS -O2 -std=gnu++11 -pthread -fPIC -shared -rdynamic)


message(STATUS "CMAKE_SYSTEM_NAME: ${CMAKE_SYSTEM_NAME}")

    set(CMAKE_THREAD_PREFER_PTHREAD TRUE)
    set(THREADS_PREFER_PTHREAD_FLAG TRUE)
    find_package(Threads REQUIRED)
    link_libraries(Threads::Threads)

include(TestBigEndian)
TEST_BIG_ENDIAN(IS_BIG_ENDIAN)
if(NOT IS_BIG_ENDIAN)
  add_definitions(-D__LITTLE_ENDIAN__)
endif()
add_definitions(-D_USE_MATH_DEFINES)


if(${CMAKE_SYSTEM_NAME} STREQUAL Linux OR ${CMAKE_SYSTEM_NAME} STREQUAL Android)
        message("Linux DETECTED!")
        if(ENABLE_ALSA)
            find_package(ALSA REQUIRED)
            if(ALSA_FOUND)
                include_directories(${ALSA_INCLUDE_DIRS})
                link_libraries(${ALSA_LIBRARIES})
                add_definitions(-D__LINUX_ALSA__)
            endif()
        endif()
endif()
# Try to find PkgConfig
find_package(PkgConfig QUIET)

if(PkgConfig_FOUND)
    # PkgConfig is available, use it
    pkg_check_modules(JACK QUIET jack)
endif()

if(NOT JACK_FOUND)
    # PkgConfig was not found or Jack was not found through it, try a fallback
    message(STATUS "PkgConfig not found or failed to find Jack, attempting fallback")

    # Fallback: Search in common locations
    find_path(JACK_INCLUDE_DIR
        NAMES jack/jack.h
        HINTS
            ENV JACK_ROOT
            "$ENV{ProgramFiles}/Jack"
            /usr/local/include
            /usr/include
    )

    find_library(JACK_LIBRARY
        NAMES jack
        HINTS
            ENV JACK_ROOT
            "$ENV{ProgramFiles}/Jack"
            /usr/local/lib
            /usr/lib
    )

    # Check if the fallback was successful
    if(JACK_INCLUDE_DIR AND JACK_LIBRARY)
        set(JACK_FOUND TRUE)
        set(JACK_INCLUDE_DIRS ${JACK_INCLUDE_DIR})
        set(JACK_LIBRARIES ${JACK_LIBRARY})
        message(STATUS "Found Jack (fallback):")
        message(STATUS "  Includes: ${JACK_INCLUDE_DIRS}")
        message(STATUS "  Libraries: ${JACK_LIBRARIES}")
    else()
        message(WARNING "Failed to find Jack library even with fallback. Please install Jack development package or ensure it is in a standard location.")
    endif()
endif()

if(JACK_FOUND)
    include_directories(${JACK_INCLUDE_DIRS})
    link_libraries(${JACK_LIBRARIES})
    add_definitions(-D__UNIX_JACK__)
endif()

add_definitions(
  -D_USE_MATH_DEFINES
  -D__LINUX_ALSA__
  -D__LITTLE_ENDIAN__
  -D__UNIX_JACK__
  -D_STK_DEBUG_
  -D__RTAUDIO_DEBUG__
  -D__RTMIDI_DEBUG__
  -Dstk_SHARED_EXPORTS
)

link_libraries(
  jack
  asound
)

include_directories(
  include
)

add_library(
  stk
  src/Modulate.cpp
  src/Envelope.cpp
  src/PoleZero.cpp
  src/FreeVerb.cpp
  src/BiQuad.cpp
  src/RtAudio.cpp
  src/Noise.cpp
  src/Whistle.cpp
  src/BeeThree.cpp
  src/Socket.cpp
  src/HevyMetl.cpp
  src/InetWvIn.cpp
  src/BandedWG.cpp
  src/ADSR.cpp
  src/Stk.cpp
  src/Thread.cpp
  src/TcpServer.cpp
  src/OneZero.cpp
  src/StifKarp.cpp
  src/PercFlut.cpp
  src/Asymp.cpp
  src/DelayA.cpp
  src/Chorus.cpp
  src/BlitSaw.cpp
  src/Skini.cpp
  src/FileLoop.cpp
  src/BlowBotl.cpp
  src/Sampler.cpp
  src/TapDelay.cpp
  src/SingWave.cpp
  src/TwoPole.cpp
  src/UdpSocket.cpp
  src/FormSwep.cpp
  src/Twang.cpp
  src/RtMidi.cpp
  src/MidiFileIn.cpp
  src/RtWvIn.cpp
  src/Guitar.cpp
  src/Mandolin.cpp
  src/BlitSquare.cpp
  src/Phonemes.cpp
  src/RtWvOut.cpp
  src/Bowed.cpp
  src/Clarinet.cpp
  src/BlowHole.cpp
  src/LentPitShift.cpp
  src/Iir.cpp
  src/Mutex.cpp
  src/InetWvOut.cpp
  src/Sphere.cpp
  src/PRCRev.cpp
  src/Voicer.cpp
  src/PitShift.cpp
  src/FileWrite.cpp
  src/DelayL.cpp
  src/FileWvOut.cpp
  src/Plucked.cpp
  src/FM.cpp
  src/Blit.cpp
  src/Delay.cpp
  src/OnePole.cpp
  src/Messager.cpp
  src/FMVoices.cpp
  src/FileWvIn.cpp
  src/SineWave.cpp
  src/Saxofony.cpp
  src/TwoZero.cpp
  src/Mesh2D.cpp
  src/Echo.cpp
  src/Resonate.cpp
  src/Granulate.cpp
  src/Flute.cpp
  src/Fir.cpp
  src/ModalBar.cpp
  src/Wurley.cpp
  src/Drummer.cpp
  src/TcpClient.cpp
  src/Sitar.cpp
  src/TubeBell.cpp
  src/Moog.cpp
  src/FileRead.cpp
  src/Simple.cpp
  src/Modal.cpp
  src/Shakers.cpp
  src/NRev.cpp
  src/Brass.cpp
  src/Rhodey.cpp
  src/Recorder.cpp
  src/JCRev.cpp
  src/VoicForm.cpp
)


set_target_properties(stk PROPERTIES SOVERSION 1.0)
