<template>
  <slot></slot>
</template>

<script>
import { nextTick } from 'vue';

export default {
  name: 'XLazyLoad',
  props: {
    domClass: String,
    domAttrs: Array,
    times: Number,
    hadMountedNumber: Number
  },
  emits: ['handleLazyData'],
  data() {
    return {
      lazyIns: null,
      lazyTimes: {}
    };
  },
  watch: {
    hadMountedNumber() {
      this.setObserve();
    }
  },
  methods: {
    setObserve() {
      nextTick(() => {
        Array.from(document.querySelectorAll(`.${this.domClass}`)).forEach((dom) => {
          this.lazyIns.observe(dom);
        });
      });
    },
    setTimes(lazyId) {
      if (!this.lazyTimes[lazyId]) {
        this.lazyTimes[lazyId] = 1;
      } else {
        this.lazyTimes[lazyId] += 1;
      }
    },
  },
  mounted() {
    nextTick(() => {
      this.lazyIns = new IntersectionObserver((changes) => {
        changes.forEach((change) => {
          if (change.intersectionRatio > 0) {
            const attrValues = {};
            const targetDom = change.target.attributes;
            // 只记录有唯一标识符的dom
            if (targetDom.lazyid && targetDom.lazyid.value) {
              const lazyId = targetDom.lazyid.value;
              // 获取当前记录的次数
              let loadTimes = 0;
              if (this.lazyTimes[lazyId]) {
                loadTimes = this.lazyTimes[lazyId];
              }
              if (loadTimes < this.times) {
                this.setTimes(lazyId);
                attrValues[lazyId] = {};
                this.domAttrs.forEach((attr) => {
                  let curValue = '';
                  if (targetDom[attr]) {
                    curValue = targetDom[attr].value;
                  }
                  attrValues[lazyId][attr] = curValue;
                });
                this.$emit('handleLazyData', attrValues);
              }
            }
          }
        });
      });
    });
  }
};
</script>
