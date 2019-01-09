<template>
    <div class="bottom">
        <input type="file" @change="handleWalletSelect"/>
        <input type="password" v-model="wallet_pass"/>
        <button
            class="btn btn-lg btn-primary btn-block"
            type="submit"
            @click="decryptBackup"
        >{{ $t('next_btn') }}</button>
        <b-modal
            id="loaderAnim"
            ref="loaderAnimModal"
            centered
            no-close-on-esc
            no-close-on-backdrop
            hide-header
            hide-header-close
            hide-footer
            title="Loading..."
        >
            <div class="lds-roller"><div /><div /><div /><div /><div /><div /><div /><div /></div>
        </b-modal>
        <b-modal
            id="error"
            ref="errorModal"
            centered
            hide-footer
            :title="$t('error_lbl')"
        >
            {{ errorMsg }}
        </b-modal>
        
        <p class="mt-2 mb-2 small">&copy; 2018 BitShares</p>
    </div>
</template>

<script>
    import LangSelect from "./lang-select";
    import {PrivateKey,PublicKey,Aes} from "bitsharesjs";
    import {compress, decompress} from "lzma";    
    import { Apis } from "bitsharesjs-ws";

    export default {
        name: "ImportBin",
        i18nOptions: { namespaces: "common" },
        components: { LangSelect },
        data() {
            return {
                wallet_file: null,
                wallet_pass: null,
                errorMsg: ''
            };
        },
        methods: {
            handleWalletSelect: function(e) {
                this.wallet_file=e.target.files[0];
            },
            decryptBackup: function() {
                this.$refs.loaderAnimModal.show();
                let reader = new FileReader();
                reader.onload = evt => {
                    let backup_buffer = new Buffer.from(evt.target.result, "binary");                
                    let private_key=PrivateKey.fromSeed(this.wallet_pass);
                    let public_key = PublicKey.fromBuffer(backup_buffer.slice(0, 33));
                    backup_buffer = backup_buffer.slice(33);
                    backup_buffer = Aes.decrypt_with_checksum(
                        private_key,
                        public_key,
                        null /*nonce*/,
                        backup_buffer
                    );
                    decompress(backup_buffer, async (wallet_string) => {
                        try {
                            let wallet_object = JSON.parse(wallet_string);                            
                            this.$refs.loaderAnimModal.hide();
                            let refs=[];
                            for(let i=0;i<wallet_object.private_keys.length;i++) {
                                refs.push(wallet_object.private_keys[i].pubkey);
                            }
                            await Apis.instance(
                                    this.$store.state.SettingsStore.settings.selected_node,
                                    true
                                ).init_promise.then(() => {
                                    return Apis.instance()
                                    .db_api()
                                    .exec("get_key_references", [refs])
                                    .then(res => {                                   
                                        console.log(res);     
                                    });
                                });
                        } catch (error) {
                            if (!wallet_string) wallet_string = "";
                            console.error(
                                "Error parsing wallet json",
                                wallet_string.substring(0, 10) + "..."
                            );
                            //reject("Error parsing wallet json");
                        }
                    });
                };
                reader.readAsBinaryString(this.wallet_file);
            }
        }
    };
</script>