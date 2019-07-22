<template>
  <section class="app-wrapper">
    <section class="sidebar">
      <div class="form">
        <label for="file-upload" class="custom-file-upload">
          <i class="fa fa-cloud-upload"></i> Upload
          {{selectedPics$}}
        </label>
        <input id="file-upload" type="file" ref="file" @change="handleFileUpload()" name="upload" />
      </div>
      <div class="assets">
        <div class="images">
          <img 
          v-for="(image, index) in images$" 
          :key="index" 
          :src="image"
          v-stream:click="{subject:clickOnThumbnails$, data:image}"
          class="images-thumbnail" />
        </div>
      </div>
    </section>
    <section class="canvas">
      <!-- v-for for all the text and images -->
      <!-- text and images will be a object their respactive position in it -->
      <img
        ref="dragimg"
        :src="canvasImage$"
        class="images-thumbnail"
      />
    </section>
  </section>
</template>

<script>
import { merge, from, of, fromEvent } from 'rxjs'
import {
  map,
  pluck,
  switchMap,
  mapTo,
  mergeMap,
  catchError,
  share,
  startWith,
  combineLatest,
  takeUntil,
} from 'rxjs/operators'
import './style.scss'
export default {
  domStreams: ['clickOnThumbnails$'],
  data() {
    return {
      file: null,
    }
  },
  data() {
    return {
      file: null,
      selectedImages:[],
    }
  },
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0]
      console.log('handleFileUpload-->', this.file)
    },
  },
  subscriptions() {
    const log = x => console.log(x)
    const move$ = fromEvent(document, 'mousemove')
    const down$ = fromEvent(document, 'mousedown')
    const up$ = fromEvent(document, 'mouseup')
    let img = []
    const canvasImage$ = this.clickOnThumbnails$.pipe(
      pluck('data'),
      startWith('http://localhost:8000/images/uploads-1462948470584.png'))
    const selectedPics$ = (imageURL) => imageURL => of([...this.selectedImages, imageURL])
    const track$ = down$.pipe(mergeMap(down => move$.pipe(takeUntil(up$))))
    const selectedImages$ = canvasImage$.subscribe(selectedPics$)
    // track$.subscribe(log)
    // d$.subscribe(log)

    const file$ = this.$watchAsObservable('file', {
      immediate: false,
    }).pipe(pluck('newValue'))

    const uploadFile = file => {
      let formData = new FormData()
      formData.append('upload', file)
      return from(
        fetch('http://localhost:8000/uploads', {
          method: 'POST',
          body: formData,
        })
          .then(res => res.json())
          .catch(err => console.log(err))
      )
    }

    const createLoader = url => from(fetch(url).then(res => res.json()))
    const images$ = createLoader('http://localhost:8000/images')
    const uploadImage$ = file$.pipe(switchMap(uploadFile))
    return {
      file$,
      images$,
      uploadImage$,
      canvasImage$,
      selectedImages$,
      selectedPics$,
    }
  },
}
</script>
