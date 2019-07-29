<template>
  <gallery-item class="gallery-item-image">
    <div class="gallery-item-info p-3">
      <a v-if="downloadUrl" class="icon download" :href="downloadUrl" title="Download">
        <icon type="download" view-box="0 0 20 22" width="16" height="16"/>
      </a>
      <a v-if="removable" class="icon delete" href="#" @click.prevent="$emit('remove')" title="Remove">
        <icon type="delete" view-box="0 0 20 20" width="16" height="16"/>
      </a>
      <a v-if="isCustomPropertiesEditable" class="icon edit" href="#" @click.prevent="$emit('edit-custom-properties')" title="Edit custom properties">
        <icon type="edit" view-box="0 0 20 20" width="16" height="16"/>
      </a>
      <a class="preview" href="#" @click.prevent="showPreview">
        <icon type="search" view-box="0 0 20 20" width="30" height="30"/>
      </a>
      <a v-if="croppable" class="icon crop" href="#" @click.prevent="$emit('crop-start', image)">
        <scissors-icon brand="var(--info)" view-box="0 0 20 20" width="16" height="16"/>
      </a>
    </div>
    <img :src="src" :alt="image.name" class="gallery-image" />
      <div v-if="image.hasOwnProperty('custom_properties')" class="image-status">
          <template v-if="image.custom_properties.status == 0">
            <span class="status status__undefined"></span>
          </template>
          <template v-if="image.custom_properties.status == 1">
            <span class="status status__approved"></span>
          </template>
          <template v-if="image.custom_properties.status == 2">
            <span class="status status__disapproved"></span>
          </template>

          <div v-if="image.custom_properties.hasOwnProperty('comment') && image.custom_properties.comment != null && image.custom_properties.comment.length > 0">
              <img style="filter: invert(1);" width="18" height="18" src="data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAABgAAAAYCAMAAADXqc3KAAAAw1BMVEVHcEwAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABoCuaQAAAAQHRSTlMANPfdZ/As6Q8EKPHSrPT+7nxDhs6v/Jo4yM2ujw4Tep7mjtFvDTEXO0ACywfvsTCXNRY6fT/Fz5Ev1P2b3qDEnRcG6AAAAL5JREFUKM+dkucSgyAMgHG17mr33nvvPXj/p2oNomC5613zx/h9kAsEhP4NXS0orqvUVJ3Dxl7DYSx8I+ajI2bitY3K5DEX1UYo2jgRd8KHGQqymzR8bysQ52glQmYHki6IKyNQCvbYIKwg3dFOpOBPgxSOMOaEDKn9XaoHos4IwvEaxCVqdyIRjg+kbCl5wBm9khPPi2XaSbPCiRxz7S0r5sspO5GB6jw8L+DOXDDHD38aogFjuS+evG/+fh1vWFc8wc2fcdsAAAAASUVORK5CYII=">
<!--              {{ image.custom_properties.comment }}-->
          </div>
      </div>
  </gallery-item>
</template>

<script>
  import ScissorsIcon from './icons/Scissors';
  import GalleryItem from './GalleryItem';

  export default {
    components: {
      ScissorsIcon,
      GalleryItem,
    },
    props: ['image', 'field', 'removable', 'editable', 'isCustomPropertiesEditable'],
    data: () => ({
      acceptedMimeTypes: ['image/jpg', 'image/jpeg', 'image/png'],
      src: "data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==",
    }),
    computed: {
      downloadUrl() {
        return this.image.id ? `/nova-vendor/ebess/advanced-nova-media-library/download/${this.image.id}` : null;
      },
      croppable() {
        return this.editable &&
          this.field.croppable &&
          this.acceptedMimeTypes.includes(this.image.mime_type || this.image.file.type);
      },
    },
    watch: {
      image: {
        handler: 'getImage',
        immediate: true
      }
    },
    methods: {
      showPreview() {
        const blobUrl = this.image.file ? URL.createObjectURL(this.image.file) : this.image.__media_urls__.preview;
        window.open(blobUrl, '_blank');
      },
      getImage() {
        if (this.editable && this.image.__media_urls__.form) {
          this.src = this.image.__media_urls__.form;
          return;
        }

        if (!this.editable && this.image.__media_urls__.detailView) {
          this.src = this.image.__media_urls__.detailView;
          return;
        }

        if (this.isVideo(this.image.__media_urls__.__original__)) {
          //Seconds to seek to, to get thumbnail of video
          let seconds = 1;  //TODO get this from the field instead of hardcoding it here
          this.getVideoThumbnail(this.image.__media_urls__.__original__, seconds);
          return;
        }

        this.src = this.image.__media_urls__.__original__;
      },
      getVideoThumbnail(path, secs = 0) {
        const video = document.createElement('video');
        video.onloadedmetadata = () => {
          video.currentTime = Math.min(Math.max(0, (secs < 0 ? video.duration : 0) + secs), video.duration);
        };
        video.onseeked = (e) => {
          const canvas = document.createElement('canvas');
          canvas.height = video.videoHeight;
          canvas.width = video.videoWidth;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(video, 0, 0, canvas.width, canvas.height);
          this.src = canvas.toDataURL();
        };
        video.src = path;
      },
      isVideo(mediaPath) {
        if (mediaPath.startsWith("data:video"))
          return true;
        //TODO better video detection
        const supportedExtensions = [".mp4", ".ogv", ".webm"];
        return supportedExtensions.some((suffix) => {
          return mediaPath.endsWith(suffix)
        });
      }
    },
  };
</script>

<style lang="scss">
  $bg-color: #e8f5fb;
  $item-max-size: 150px;
  $border-radius: 10px;

  .gallery {
      .gallery-item {
          border: 1px solid gray;
          background: #fff;
      }
    .gallery-item-image.gallery-item {
      width: $item-max-size;
      height: $item-max-size;
      padding-top: 60px;

      &:hover .gallery-item-info {
        display: flex;
        background-color: rgba(23, 44, 53, 0.8);
      }

      .gallery-item-info {
        display: none;
        align-items: center;
        justify-content: center;
        flex-direction: column;
        background-color: transparentize($bg-color, .2);
        border-radius: $border-radius;
        position: absolute;
        z-index: 10;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;

        .preview {
          color: var(--black);
        }

        .delete {
          right: 10px;
          color: var(--danger);
        }

        .crop {
          left: 10px;
          top: auto;
          bottom: 10px;
        }
      }

      .gallery-image {
        object-fit: contain;
        display: block;
        max-height: 100%;
        border-radius: $border-radius;
      }
    }

    .icon {
      cursor: pointer;
      position: absolute;
      top: 10px;
      color: var(--info);
    }

    .edit {
      right: 30px;
    }

    .download {
      left: 10px;
    }
  }

.image-status {
    width: 100%;
    background: #4c4c4c;
    position: absolute;
    top: 0;
    left: 0;
    border-top-left-radius: 5px;
    border-top-right-radius: 5px;
    padding: 5px 10px;
    color: white;
    min-height: 32px;

    .status {
        position: absolute;
        width: 10px;
        height: 10px;
        display: block;
        border-radius: 50%;
        right: 8px;
        border: 1px solid gray;

        &__undefined {
            background: gray;
        }

        &__approved {
            background: greenyellow;
        }

        &__disapproved {
            background: red;
        }
    }
}
</style>
