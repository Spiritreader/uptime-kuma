<template>
    <div class="wrap" :style="wrapStyle" ref="wrap">
        <div class="hp-bar-big" :style="barStyle">
            <div
                class="beat"
                :class="{ 'empty' : (beat === 0), 'down' : (beat.status === 0), 'pending' : (beat.status === 2) }"
                :style="beatStyle"
                v-for="(beat, index) in shortBeatList"
                :key="index"
                :title="beat.msg">
            </div>
        </div>
    </div>
</template>

<script>


export default {
    props: {
        size: {
            type: String,
            default: "big"
        },
        monitorId: Number
    },
    data() {
        return {
            beatWidth: 10,
            beatHeight: 30,
            hoverScale: 1.5,
            beatMargin: 3,      // Odd number only, even = blurry
            move: false,
            maxBeat: -1,
        }
    },
    unmounted() {
        window.removeEventListener("resize", this.resize);
    },
    mounted() {
        if (this.size === "small") {
            this.beatWidth = 5.6;
            this.beatMargin = 2.4;
            this.beatHeight = 16
        }

        window.addEventListener("resize", this.resize);
        this.resize();
    },
    methods: {
        resize() {
            if (this.$refs.wrap) {
                this.maxBeat = Math.floor(this.$refs.wrap.clientWidth / (this.beatWidth + this.beatMargin * 2))
            }
        }
    },
    computed: {

        beatList() {
            if (! (this.monitorId in this.$root.heartbeatList)) {
                this.$root.heartbeatList[this.monitorId] = [];
            }
            return this.$root.heartbeatList[this.monitorId]
        },

        shortBeatList() {
            let placeholders = []

            let start = this.beatList.length - this.maxBeat;

            if (this.move) {
                start = start - 1;
            }

            if (start < 0) {
                // Add empty placeholder
                for (let i = start; i < 0; i++) {
                    placeholders.push(0)
                }
                start = 0;
            }



            return placeholders.concat(this.beatList.slice(start))
        },

        wrapStyle() {
            let topBottom = (((this.beatHeight * this.hoverScale) - this.beatHeight) / 2);
            let leftRight = (((this.beatWidth * this.hoverScale) - this.beatWidth) / 2);

            let width
            if (this.maxBeat > 0) {
                width = (this.beatWidth + this.beatMargin * 2) * this.maxBeat + (leftRight * 2) + "px"
            } {
                width = "100%"
            }

            return {
                padding: `${topBottom}px ${leftRight}px`,
                width: width
            }
        },

        barStyle() {
            if (this.move && this.shortBeatList.length > this.maxBeat) {
                let width = -(this.beatWidth + this.beatMargin * 2);

                return {
                    transition: "all ease-in-out 0.25s",
                    transform: `translateX(${width}px)`,
                }

            } else {
                return {
                    transform: `translateX(0)`,
                }
            }
        },

        beatStyle() {
            return {
                width: this.beatWidth + "px",
                height: this.beatHeight + "px",
                margin: this.beatMargin + "px",
                "--hover-scale": this.hoverScale,
            }
        }

    },
    watch: {
        beatList: {
            handler(val, oldVal) {
                this.move = true;

                setTimeout(() => {
                    this.move = false;
                }, 300)
            },
            deep: true,
        }
    }
}
</script>

<style scoped lang="scss">
@import "../assets/vars.scss";

.wrap {
    overflow: hidden;
    width: 100%;
    white-space: nowrap;
}

.hp-bar-big {
    .beat {
        display: inline-block;
        background-color: $primary;
        border-radius: 50rem;

        &.empty {
            background-color: aliceblue;
        }

        &.down {
            background-color: $danger;
        }

        &.pending {
            background-color: $warning;
        }

        &:not(.empty):hover {
            transition: all ease-in-out 0.15s;
            opacity: 0.8;
            transform: scale(var(--hover-scale));
        }
    }
}

</style>
