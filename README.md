# vue-fullpage-component
```
npm i vue-fullpage-component
```

```html
<template>
<fullpage :accuracy="20" :startAt="0" duration="200">
  <section>page 1</section>
  <div>page 2</div>
  ...
</fullpage>
</template>

<script>
import fullpage from 'vue-fullpage-component'

export default {
  components: {
    fullpage,
  }
}
</script>
```
