<template>
    <div>
        <!-- <back-header /> -->
        <h1>Upload Material:</h1>
        <form enctype="multipart/form-data" novalidate>
            <div class="dropbox">
                <iput 
                    type="file" 
                    multiple 
                    :name="UploadFieldName"
                    :disabled="isSaving"
                    @change="
                        filesChange($event.target.name, $event.target.files);
                        fileCount = $event.target.files.length;
                    "
                    accept="image/*"
                    class="input-file"
                />
                <!-- <p v-if="isInitial || isSuccess"> -->
                <p v-if="isInitial">
                    Drge your file(s) here to begin<br />
                    or click to browse
                </p>
                <p v-if="isSaving">Uploading {{fileCount}} files...</p>
                <p v-if="isSuccess">Upload Successful.</p>
                <p v-if="isFailed">Upload Failed.</p>
            </div>
        </form>
    </div>
</template>
<script>
import UploadService from '@/services/UploadService';

const STATUS_INITIAL = 0,
    STATUS_SAVING = 1,
    STATUS_SUCCESS = 2,
    STATUS_FAILED = 3;

export default{
    data () {
        return {
            BASE_URL: "http://localhost:8081/assets/uploads/",
            error: null,
            uploadError: null,
            currentStatus: null,
            uploadFieldName: "userPhoto",
            uploadedFileNames: [],
        };
    },
    methods: {
        navigateTo (route) {
            console.log(route);
            this.$router.push(route);
        },
        wait(ms) {
            return (x) => {
                return new Promise((resolve) => setTimeout(() => resolve(x), ms));
            };
        },
        reset() {
            this.currentStatus = STATUS_INITIAL;
            this.uploadError = null;
            this.uploadedFileNames = [];
        },
        async save(formData) {
            try {
                this.currentStatus = STATUS_SAVING;
                await UploadService.upload(formData);
                this.currentStatus = STATUS_SUCCESS;
                this.clearUploadResult();
            } catch (error) {
                console.log(error);
                this.currentStatus = STATUS_FAILED;
            }
        },
        filesChange(fieldName, fileList) {
            const formData = new FormData();

            if (!fileList.length) return;

            Array.form(Array(fileList.length).keys()).map(x => {
                formData.append(fieldName, fileList[x], fileList[x].name);
                this.uploadedFileNames.push(fileList[x].name);
            });
            this.save(formData);
        },
        clearUploadResult: function(){
            setTimeout(() => this.reset(), 5000);
        },
    },
    created(){
        this.reset();
    },
    computed: {
        isInitial() {
            return this.currentStatus === STATUS_INITIAL;
        },
        isSaving() {
            return this.currentStatus === STATUS_SAVING;
        },
        isSuccess() {
            return this.currentStatus === STATUS_SUCCESS;
        },
        isFailed() {
            return this.currentStatus === STATUS_FAILED;
        }
    }
};
</script>

<style scoped>
    .dropbox {
        outline: 2px dashed grey; /*the dash box */
        outline-offset: -10px;
        background:lemonchiffon;
        color: dimgray;
        padding: 10px 10px;
        min-height: 200px;
        position: relative;
        cursor: pointer;
    }
    .input-file {
        opacity: 0; 
        width: 100%;
        height: 200px;
        position: absolute;
        cursor: pointer;
    }
    .dropbox:hover {
        background:khaki;
    }
    .dropbox p {
        font-size: 1.2em;
        text-align: center;
        padding: 50px 0;
    }
</style>