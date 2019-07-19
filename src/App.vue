<template>
  <section>
    <section class="sidebar">
      {{file$}}
      <div class="form">
        <label for="file-upload" class="custom-file-upload">
          <i class="fa fa-cloud-upload"></i> Upload
        </label>
        <input id="file-upload" type="file" ref="file" @change="handleFileUpload()" name="upload"/>
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
  methods: {
    handleFileUpload() {
      this.file = this.$refs.file.files[0]
      console.log('handleFileUpload-->', this.file)
    },
  },
  subscriptions() {
    const file$ = this.$watchAsObservable('file', {
      immediate: false,
    }).pipe(pluck('newValue'))
    // const prepareFile = file => {
    //   console.log('prepareFile-->', file)
    //   const formData = new FormData()
    //   formData.append('file', file)
    //   return formData
    // }
    const uploadFile = file => {
      console.log('uploadfile file -->', file)
      let formData = new FormData()
      formData.append('upload', file)
      console.log('uploadFile-->', formData)
          for (var key of formData.entries()) {
        console.log(key[0] + ', ' + key[1]);
    }
      return from(
        fetch('http://localhost:8000/uploads', {
          method: 'POST',
          body: formData,
        }).then(res=>res.json()).catch(err=>console.log(err))
      )
    }

    const createLoader = url => from(fetch(url).then(res=>res.json()))
    const images$ = createLoader('http://localhost:8000/images')
    const uploadImage$ = file$.pipe(
      switchMap(uploadFile)
    )
    return {
      file$,
      images$,
      uploadImage$,
    }
  },
}
</script>
