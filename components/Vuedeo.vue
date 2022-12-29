<template>
    <div class="bg">
    <div class="video-container" ref="videoContainer" @mouseleave="hideControls" @mousemove="showControlsNoTimeout"
    :class="cursorClass">
        <video :src="`test.mp4#t=${$route.query.t}`"
        class="video" ref="video"
        @timeupdate="updateTime"
        @click="play"></video>

        <div class="controls" :class="controlsClass">
            <div class="bottom-controls">
                <div class="hover-time" :style="hoverTimeStyle" 
                :class="hoverTimeClass">{{hoverTime}}</div>

                <input type="range" min="0" max="100"
                class="slider" v-model="slider" 
                @input="changeCurrentTime"
                @mouseenter="showHoverTime"
                @mousemove="position"
                @mouseleave="hideHoverTime"
                @keyup.space="play">
                <div class="visible-slider">
                    <div :style="visibleSliderStyle" ref="visibleSliderThumb" 
                    class="visible-slider-thumb">
                    </div>
                </div>
            </div>
            <div class="top-controls">
                <div class="left-controls">
                    <div @click="play" class="play-btn control-icon">
                        <span v-if="isPaused" class="material-symbols-outlined">
                            play_arrow
                        </span>
                        <span v-else class="material-symbols-outlined">
                            pause
                        </span>
                    </div>
                    <div class="vol-btn control-icon">

                        <span @click="mute" v-if="isMuted" class="material-symbols-outlined">
                            volume_off
                        </span>
                        <span @click="mute" v-else class="material-symbols-outlined">
                            volume_up
                        </span>
                        <input class="volume" type="range" min="0" max="100"
                        @input="changeVolume"
                        v-model="volume">

                    </div>
                </div>
                <div class="right-controls">

                    <div class="copy-link control-icon" 
                    title="Copy timestamp link"
                    @click="copyLink">
                        <span class="material-symbols-outlined">
                            link
                        </span>
                    </div>
                    <div class="timer blur-bg">
                        {{timer}} / {{duration}}
                    </div>

                    <div class="fullscreen-btn control-icon" @click="toggleFullscreen">
                        <!-- <span v-if="isFullscreened" class="material-symbols-outlined">
                            fullscreen_exit
                        </span> -->
                        <span class="material-symbols-outlined">
                            fullscreen
                        </span>
                    </div>
                </div>
            </div>
        </div>
    </div>
    </div>
</template>

<script setup>
const video = ref(null),
slider = ref(0),
timer = ref("00:00"),
isMuted = ref(false),
isPaused = ref(true),
isFullscreened = ref(false),
videoContainer = ref(null),
controlsClass = ref(""),
duration = ref("00:00"),
visibleSliderThumb = ref(null),
visibleSliderStyle = ref(""),
hoverTime = ref("0"),
hoverTimeStyle = ref(""),
hoverTimeClass = ref("hide"),
cursorClass = ref(""),
volume = ref(80)


onMounted(() => {
    if (!video.value) return
    duration.value = formatTime(video.value.duration)
})

function copyLink() {
    navigator.clipboard.writeText(
        `${location.href.replace(location.search, '')}?t=${parseInt(video.value.currentTime)}`
    )
}


function changeVolume() {
    video.value.volume = volume.value / 100
    if (video.value.volume == 0) {
        isMuted.value = true
        video.value.muted = true
    } else {
        isMuted.value = false
        video.value.muted = false
    }
}

function hideHoverTime() {
    hoverTimeClass.value = "hide"
}

function showHoverTime() {
    hoverTimeClass.value = "show"
}

function position(e) {
    const x = e.pageX
    const l = e.target.getBoundingClientRect().left
    const w = e.target.parentNode.getBoundingClientRect().width
    const pt = (x - l) / w
    if (x<l || x>(l+w)) return
    const res = formatTime(video.value.duration * pt)
    hoverTime.value = res
    hoverTimeStyle.value = `left: ${x - l - 20}px`

}

function hideControls() {
    if (isPaused.value === true) return
    controlsClass.value = "transparent"
    // cursorClass.value = "hidden-cursor"
}

function showControls() {
    controlsClass.value = ""
    cursorClass.value = ""
    if (isPaused.value === true) return
    timeout()
}

function timeout() {
    setTimeout(() => {
        // controlsClass.value = "transparent"
        // cursorClass.value = "hidden-cursor"
    }, 2000)
}

function showControlsNoTimeout() {
    controlsClass.value = ""
    cursorClass.value = ""
    cursorClass.value = ""
}

function changeCurrentTime() {
    const c = (video.value.duration * slider.value) / 100
    video.value.currentTime = c
    timer.value = formatTime(video.value.currentTime)
    visibleSliderStyle.value = `width: ${slider.value}%`

}

function formatTime(x) {
    x = parseInt(x)
    const min = parseInt(x / 60)
    const sec = x - (min * 60)
    if (x < 10) return `00:0${x}`
    if (x >= 10 && x < 60) return `00:${x}`
    if (min < 10 && sec < 10) return `0${min}:0${sec}`
    if (min < 10 && sec >= 10) return `0${min}:${sec}`
    if (min >= 10 && sec < 10) return `${min}:0${sec}`
    if (min >= 10 && sec >= 10) return `${min}:${sec}`
    return "00:00"
}

function play() {
    isPaused.value = !isPaused.value
    if (isPaused.value === true) {
        video.value.pause()
        controlsClass.value = ""
        cursorClass.value = ""
    }
    if (isPaused.value === false) {
        video.value.play()
        controlsClass.value = "transparent"
        cursorClass.value = "hidden-cursor"
    }
}

function mute() {
    isMuted.value = !isMuted.value
    video.value.muted = isMuted.value
}

function updateTime() {
    slider.value = (video.value.currentTime / video.value.duration) * 100
    timer.value = formatTime(video.value.currentTime)
    if (video.value.currentTime >= video.value.duration) {
        isPaused.value = true
        showControlsNoTimeout()
    }
    visibleSliderStyle.value = `width: ${slider.value}%`

}

function toggleFullscreen() {
    isFullscreened.value = !isFullscreened.value
    if (isFullscreened.value === true) videoContainer.value.requestFullscreen()
    if (isFullscreened.value === false) {
        videoContainer.value.getRootNode().exitFullscreen()
    }
}

</script>



<style scoped>

@import url('https://fonts.googleapis.com/css2?family=Inter&display=swap');


.bg {
    background: black;
    width: 100%;
    height: fit-content;
    overflow: hidden;
}

.hide {
    opacity: 0;
}
.visible-slider {
    width: 100%;
    /* background: rgb(163, 163, 163); */
    /* background: rgba(0, 0, 0, 0.301); */
    height: 10px;
    position: relative;
    top: -20px;
    margin-bottom: -20px;
    border-radius: 3px;
    pointer-events: none;
    /* border-top: 1px solid rgba(255, 255, 255, 0.473); */
    padding: 5px 0;
}

/* .bottom-controls:hover .visible-slider {
    height: 15px;
} */

.visible-slider-thumb {
    height: 100%;
    width: 10px;
    background: lightgrey;
    transition: width 0.2s linear;
    border-radius: 3px;
    pointer-events: none;
}

.hover-time {
    width: fit-content;
    position: relative;
    height: fit-content;
    top: -30px;
    margin-bottom: -30px;
    text-shadow: 0px 0px 8px black;
    background: rgba(0, 0, 0, 0.301);
    padding: 5px;
    border-radius: 5px;
    /* border: 1px solid red; */
}

.hidden-cursor {
    cursor: none !important;
}

.transparent {
    opacity: 0;
}

.controls {
    padding: 2px;
    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    position: relative;
    top: -60px;
    margin-bottom: -60px;
    box-sizing: border-box;
    height: 60px;
    backdrop-filter: blur(2px);
    transition: opacity 0.6s;
    /* border-radius: 3px; */
    /* border: 1px solid red; */

}

.top-controls {
    width: 100%;
    display: flex;
    justify-content: space-between;
}

.left-controls, .right-controls {
    display: flex;
    justify-content: center;
    align-items: center;
}
.bottom-controls {
    width: 100%;
    display: flex;
    flex-direction: column;
}

.video-container {
    width: 100%;
    background: black;
    display: flex;
    flex-direction: column;
    max-width: 80%;
    margin: auto;
    user-select: none;
    /* overflow: hidden; */
    color: rgb(230, 230, 230);
    font-family: 'Inter', sans-serif;
    max-height: 100%;

}
.video {
    max-height: 100%;
}

.slider {
    width: 100%;
    -webkit-appearance: none;
    appearance: none;
    height: 12px;
    outline: none;
    background: transparent;
}

.control-icon {
    width: fit-content;
    height: fit-content;
    display: flex;
    justify-content: center;
    align-items: center;
    border-radius: 50px;
    /* backdrop-filter: blur(10px); */
    cursor: default;
    /* background: black; */
    /* text-shadow: 0px 0px 8px black; */
    background: rgba(0, 0, 0, 0.301);
    padding: 5px;
    border-radius: 5px;
}

.control-icon:hover {
    cursor: pointer;
}

.control-icon:active {
    text-shadow: 0px 0px 4px whitesmoke;
}


.slider::-webkit-slider-thumb {
    appearance: none;
    -webkit-appearance: none;
    background: transparent;
    width: 2px;
    height: 5px;
    z-index: 100;
}


.slider::-webkit-slider-runnable-track {
    appearance: none;
    -webkit-appearance: none;
    background: transparent;
    z-index: 100;
    /* border-radius: 2px; */
    height: 10px;
}

.blur-bg {
    backdrop-filter: blur(10px);
}

.timer {
    width: fit-content;
    height: fit-content;
    display: flex;
    justify-content: center;
    align-items: center;
    /* text-shadow: 0px 0px 8px black; */
    background: rgba(0, 0, 0, 0.301);
    padding: 7px;
    border-radius: 5px;
    margin: 0 5px;
}


.material-symbols-outlined {
    /* font-size: 1.7em; */
}

.volume {
    width: 0px;
    transition: width 0.3s ease, opacity 0.3s ease;
    -webkit-appearance: none;
    appearance: none;
    background: transparent;
    outline: none;
    border: none;
    opacity: 0;
}

.volume::-webkit-slider-thumb {
    -webkit-appearance: none;
    appearance: none;
    background: lightgrey;
    width: 10px;
    height: 6px;
    border-radius: 0px;
}

.volume::-webkit-slider-runnable-track {
    appearance: none;
    background: transparent;
    border: 2px solid lightgrey;
    border-radius: 2px;
    height: 10px;
    box-sizing: border-box;
}

.vol-btn:hover .volume {
    width: 100px;
    opacity: 1;
}

.vol-btn {
    margin: 0 5px;
}

</style>