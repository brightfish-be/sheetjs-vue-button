<template>
    <button @click.prevent="initExport" class="btn btn-export" :disabled="disabled">
        <svg width="30" height="30" viewBox="0 0 24 24"
             fill="none"
             stroke="#fff"
             stroke-width="1.5"
             stroke-linecap="round">
            <path d="M8 17l4 4 4-4M12 12v9"></path>
            <path d="M20.88 18.09A5 5 0 0 0 18 9h-1.26A8 8 0 1 0 3 16.29"></path>
        </svg>
    </button>
</template>

<script>
    /**
     * SheetJS version
     * @see https://cdnjs.com/libraries/xlsx
     * @type {string}
     */
    const VERSION = '0.15.1';

    export default {
        props: {
            disabled: Boolean,
            title: String,
            file: String,
            transformer: Function
        },
        data() {
            return {
                sheetTitle: this.title || 'Export',
                filename: this.file || 'export'
            }
        },
        methods: {
            /**
             * Launch the export flow
             * @return void
             */
            initExport() {
                this.spin(1)
                    .loadScript(VERSION)
                    .then(() => this.exportToExcel().spin(0))
                    .catch(error => console.error('Script loading failed!', error))
            },
            /**
             * Export to Excel using SheetJS
             * @return this
             */
            exportToExcel() {
                let workBook = XLSX.utils.book_new(),
                    sheet = this.transformer(),
                    filename = this.getFileName(this.filename + '-' + this.sheetTitle);

                //return this;
                sheet = XLSX.utils.aoa_to_sheet(sheet);
                XLSX.utils.book_append_sheet(workBook, sheet, this.sheetTitle);

                XLSX.writeFile(workBook, filename);

                return this;
            },
            /**
             * Load the SheetJS script from a CDN given its version
             * @param {String} version
             * @param {Object} options
             * @return {Promise}
             */
            loadScript(version, options = {}) {

                if (window.XLSX) return Promise.resolve();

                return new Promise((resolve, reject) => {
                    let d = document,
                        url = `https://cdnjs.cloudflare.com/ajax/libs/xlsx/${version}/xlsx.full.min.js`,
                        ref = d.getElementsByTagName('script')[0],
                        script = d.createElement('script');

                    if (options.attributes) {
                        Object.keys(options.attributes).forEach(k => script.setAttribute(k, options.attributes[k]));
                    }

                    script.async = true;
                    script.onload = () => resolve();
                    script.onerror = error => reject(error);
                    script.src = url;

                    ref.parentNode.insertBefore(script, ref);
                })
            },
            /**
             * Setup the filename
             * @param {String} name
             * @param {Boolean} timestamp
             * @return {string}
             */
            getFileName(name, timestamp = true) {
                name = name.toLowerCase().replace(' ', '-');
                return name + (timestamp ? '-' + Math.round((new Date).getTime() / 1000) : '') + '.xlsx'
            },
            /**
             * Toggle some spinning
             * @param {Boolean|Number} state
             * @return this
             */
            spin(state) {
                this.$el.classList[state ? 'add' : 'remove']('btn-spin');
                return this;
            }
        }
    }
</script>

<style>
    .btn-spin:after {
        content: '';
        position: absolute; top: -5px; left: -5px;
        width: 50px; height: 50px;
        transform-origin: 50% 50%;
        border: 5px solid rgba(0, 0, 0, 0);
        border-top: 5px solid #fff;
        border-radius: 50%;
        opacity: 0;
        animation: spin 1s cubic-bezier(0.46, 0.03, 0.52, 0.96) infinite;
    }

    @keyframes spin {
        0% {
            opacity: 1;
            transform: rotate(0deg);
        }
        100% {
            opacity: 1;
            transform: rotate(360deg);
        }
    }
</style>
