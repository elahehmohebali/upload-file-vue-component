<template>
    <div>
        <div class="e-upload" :class="{ 'e-upload--link': link, 'e-upload--single': single }">
            <ul class="e-upload--list">
                <template v-if="typeof files == 'string'">
                    <li v-if="files" class="e-upload--item">
                        <slot name="file" :file="files"></slot>
                        <el-popconfirm :confirm-button-text="deleteAcceptText" :cancel-button-text="deleteCancelText" icon="el-icon-info" icon-color="red" :title="deleteConfirmText" @confirm="$emit('delete', { file: files, done: () => removeItem() })">
                            <template #reference>
                                <base-button v-if="!noDelete" :disabled="!editable" type="danger" link icon class="e-upload--remove">
                                    <i class="tim-icons icon-simple-remove"></i>
                                </base-button>
                            </template>
                        </el-popconfirm>
                    </li>
                </template>
                <template v-else>
                    <li v-for="(item,i) in files" :key="i" class="e-upload--item">
                        <slot name="file" :file="item"></slot>
                        <el-popconfirm :confirm-button-text="deleteAcceptText" :cancel-button-text="deleteCancelText" icon="el-icon-info" icon-color="red" :title="deleteConfirmText" @confirm="$emit('delete', { file: item, index: i, done: () => removeItem(i) })">
                            <template #reference>
                                <base-button v-if="!noDelete" :disabled="!editable" type="danger" link icon class="e-upload--remove">
                                    <i class="tim-icons icon-simple-remove"></i>
                                </base-button>
                            </template>
                        </el-popconfirm>
                    </li>
                </template>
                <li v-if="progress >= 0" class="e-upload--item e-upload--progress">
                    <el-progress type="circle" :percentage="progress" :width="80"></el-progress>
                </li>
            </ul>
            <div class="e-upload--button-container" v-if="progress < 0 && (typeof files != 'string' || !files)">
                <base-button :link="link" class="e-upload--button" type="primary" @click.stop="$refs.fileInput.click()">
                    <i :class="[ uploadIcon ]"></i>
                    <span>{{ uploadText }}</span>
                </base-button>
                <input ref="fileInput" type="file" class="e-upload--input" :accept="accept" @change="fileSelected" />
            </div>
        </div>
        <div class="e-upload--error" v-if="error">{{error}}</div>
    </div>    
</template>

<script>
export default {
    model: {
        prop: 'files',
        event: 'update'
    },
    props: { 
        error:       { type: String,  default: null },
        link:        { type: Boolean, default: false },
        files:       { type: [String,Array], default: () => [] },
        single:      { type: Boolean, default: false },
        accept:      { type: String,  default: 'image/png, image/jpeg' },
        minSize:     { type: [String,Number], default: 1024 },
        maxSize:     { type: [String,Number], default: 104857600 },
        editable:    { type: Boolean, default: true },
        noDelete:    { type: Boolean, default: false },
        uploadText:  { type: String,  default: 'Select File' },
        uploadIcon:  { type: String,  default: 'tim-icons icon-cloud-upload-94' },
        deleteConfirmText: { type: String, default: 'Are you sure you want to delete this photo?' },
        deleteCancelText:  { type: String, default: 'No' },
        deleteAcceptText:  { type: String, default: 'Yes' },   
    },
    data() {
        return {
            progress: -1,
        };
    },
    methods: {
        fileSelected (e) {
            if(this.minSize !== null && !isNaN(this.minSize)){
                for(let f of e.target.files){
                    if(f.size < Number(this.minSize)){
                        this.$message({
                            showClose: true,
                            message: `Image size is less than ${this.$utils.convertBytes(this.minSize)}!`,
                            type: 'error',
                        });
                        return;
                    }
                }
            }
            if(this.maxSize !== null && !isNaN(this.maxSize)){
                for(let f of e.target.files){
                    if(f.size > Number(this.maxSize)){
                        this.$message({
                            showClose: true,
                            message: `Image size is greater than ${this.$utils.convertBytes(this.maxSize)}!`,
                            type: 'error',
                        });
                        return;
                    }
                }
            }
            this.$emit('select', {
                selection: e.target.files,
                progress: (current) => {
                    this.progress = current;
                },
                done: (file) => {
                    this.progress = -1;
                    if(this.single){
                        this.$emit('update', file);
                    }else{
                        let val = this.files.slice();
                        val.push(file);
                        this.$emit('update', val);
                    }
                },
            });
        },
        removeItem (index) {
            if(this.single){
                this.$emit('update', '');
            }else{
                let val = this.files.slice();
                if(index >= 0 && index < val.length){
                    val.splice(index, 1);
                    this.$emit('update', val);
                }
            }
        },
    },
}
</script>

<style scoped>
    .e-upload {
        display: flex;
        flex-direction: row;
        background: #D0D0D0;
        align-items: center;
        padding: 4px;
        border-radius: 8px;
    }
    .e-upload--single {
        display: inline-block;
    }
    .e-upload--link {
        background: transparent;
    }
    .e-upload--list {
        display: flex;
        flex-direction: row;
        padding: 0px;
        margin: 0px;
    }
    .e-upload--item {
        display: flex;
        flex-direction: column;
        justify-content: center;
        position: relative;
        width: auto;
        height: auto;
        border: 1px solid #A0A0A0;
        border-radius: 8px;
        padding: 0px;
        margin: 4px;
        background: #FFFFFF;
        overflow: hidden;
        box-shadow: 2px 2px 2px rgba(0,0,0,0.1);
        transition: all 1s;
    }
    .e-upload--link .e-upload--item {
        width: auto;
        height: auto;
        padding:0 15px 0 0;
    }
    .e-upload--progress {
        align-items: center;
        justify-content: center;
    }
    .e-upload--item.removed {
        transform: translateY(100%);
        opacity: 0;
    }
    .e-upload--button {
        width: 96px;
        height: 96px;
        flex-direction: column;
        margin: 4px;
    }
    .e-upload--button > i {
        font-size: 24px;
        margin-bottom: 6px;
    }
    .e-upload--button.btn-link {
        padding: 0px;
        width: auto;
        height: auto;
    }
    .e-upload--remove {
        position: absolute;
        top: 4px;
        right: 4px;
        margin: 0px;
        width: auto;
        height: auto;
        min-width: 0px;
        border-radius: 4px;
        padding: 2px;
        background: rgba(0, 0, 0, 0.2);
    }
    .e-upload--remove > i {
        position: static !important;
        padding: 0px;
        margin: 0px;
        width: 16px !important;
        height: 16px !important;
        transform: translate(0px, 0px) !important;
    }
    .e-upload--remove > i::before {
        display: block;
        width: 16px;
        height: 16px;
        line-height: 16px;
        font-size: 16px;
    }
    .e-upload--link .e-upload--remove {
        padding: 2px;
    }
    .e-upload--link .e-upload--remove > i {
        width: 10px !important;
        height: 10px !important;
    }
    .e-upload--link .e-upload--remove > i::before {
        width: 10px;
        height: 10px;
        line-height: 10px;
        font-size: 10px;
    }
    .e-upload--input {
        position: fixed;
        left: -1000px;
        top: -1000px;
        visibility: hidden;
    }
    .e-upload--error {
        color:#ec250d;
        padding-left: 9px;
    }
</style>