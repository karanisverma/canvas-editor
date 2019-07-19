<template>
  <section>
    <section class="sidebar">
      {{file$}}
      <div class="form">
        <label for="file-upload" class="custom-file-upload">
          <i class="fa fa-cloud-upload"></i> Upload
        </label>
        <input id="file-upload" type="file" ref="file" @change="handleFileUpload" />
      </div>
      <div class="assets">
        <div class="images">
          <img v-for="(image, index) in images$" :key="index" :src="image" class="images-thumbnail" />
        </div>
      </div>
    </section>
    <section class="canvas"></section>
  </section>
</template>

<script>
import { merge, from, of } from 'rxjs'
import {
  map,
  pluck,
  switchMap,
  mapTo,
  catchError,
  share,
  startWith,
  combineLatest,
} from 'rxjs/operators'
import './style.scss'
export default {
  domStreams: ['fileUploadChange$'],
  data() {
    return {
      file: null,
    }
  },
  data() {
    return {
      file: null,
    }
  },
  methods:{
    handleFileUpload(){
      this.file = this.$refs.file.files[0]
    }
  },
  subscriptions() {
    const file$ = this.$watchAsObservable("file",{
      immediate: false
    }).pipe(pluck('newValue'))
    const prepareFile = file => {
      console.log('prepareFile-->', file)
      const formData = new FormData()
      formData.append('file', file)
      return formData
    }
    const uploadFile = formData =>
      from(
        this.$http.post('http://localhost:8000/uploads', formData, {
          headers: {
            'Content-Type': 'multipart/form-data',
          },
        })
      ).pipe(pluck('data'))

    const createLoader = url => from(this.$http.get(url)).pipe(pluck('data'))
    const images$ = createLoader('http://localhost:8000/images')
    const uploadImage$ = file$.pipe(
      switchMap(prepareFile),
      uploadFile
    )
    return {
      file$,
      images$,
      uploadImage$,
    }
  },
}
</script>
