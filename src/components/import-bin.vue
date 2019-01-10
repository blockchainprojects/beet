<template>
    <div class="bottom">
        <input type="file" @change="handleWalletSelect" v-if="step1"/>
        <input type="password" v-model="wallet_pass" v-if="step1"/>
        <button
            class="btn btn-lg btn-primary btn-block"
            type="submit"
            @click="decryptBackup"
             v-if="step1"
        >{{ $t('next_btn') }}</button>
          <table class="table small table-striped table-sm"  v-if="step2">
            <tbody>
                <tr
                    v-for="account in accounts" 
                    :key="account.id"
                >
                    <td class="text-left">{{ account.name }}</td>
                    <td class="text-right">{{ account.id }}</td>
                </tr>
            </tbody>
        </table>
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
    import WalletHandler from "../lib/WalletHandler";

    export default {
        name: "ImportBin",
        i18nOptions: { namespaces: "common" },
        components: { LangSelect },
        data() {
            return {
                wallet_file: null,
                wallet_pass: null,
                errorMsg: '',
                step1:true,
                step2:false,
                accounts:[]
            };
        },
        methods: {
            handleWalletSelect: function(e) {
                this.wallet_file=e.target.files[0];
            },
            decryptBackup: function() {
                this.$refs.loaderAnimModal.show();
                let reader = new FileReader();
                reader.onload = async evt => {
                    let backup_buffer = new Buffer.from(evt.target.result, "binary");                
                    let wh=new WalletHandler(evt.target.result);
                    try {
                        let unlocked=await wh.unlock(this.wallet_pass);
                        if (unlocked) {
                            this.accounts=await wh.lookupAccounts();
                            console.log(this.accounts);
                            this.step1=false;
                            this.step2=true;
                            this.$refs.loaderAnimModal.hide();
                        }
                    }catch(e) {
                        console.log(e);
                    }
                };
                reader.readAsBinaryString(this.wallet_file);
            }
        }
    };
</script>