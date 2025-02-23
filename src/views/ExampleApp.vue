<template>
	<div id="example-app">
		<h2>
			This page is an example Vue application using the file picker component.
			<br>
			<img src="https://vuejs.org/images/logo.png" width="25px" style="margin-bottom: -2px;"> 🚀
			Checkout the <a href="https://github.com/eneiluj/nextcloud-webdav-filepicker/blob/master/src/exampleVue.js">source file</a>
		</h2>
		<p>
			You can try the file picker with your Nextcloud instance from this page.
			<br>
			Make sure you installed the
			<a href="https://apps.nextcloud.com/apps/webapppassword">WebAppPassword app</a>
			and added <b>{{ domainToAuthorize }}</b> as allowed origin in WebAppPassword settings.
		</p>
		<br>
		<p>
			Following fields are directly plugged to NcWebdavFilePicker component's props.
		</p>
		<h3>Authentication</h3>
		<p>
			Leave login, password and token fields empty to let the file picker open an authentication popup and get an app password (web login flow).
		</p>
		<input v-model="ncUrl" type="text" placeholder="Nextcloud address">
		<input v-model="login" type="text" placeholder="login">
		<input v-model="password" type="password" placeholder="password">
		<input v-model="accessToken" type="password" placeholder="OAuth2 access token">
		<h3>Theme color</h3>
		<label for="color">Main file picker color</label>
		<input id="color" v-model="color" type="color">
		<br>
		<label for="darkmode">Dark mode</label>
		<input id="darkmode" v-model="darkMode" type="checkbox">
		<h3>Custom button to open the file picker in "getFilesPath" mode</h3>
		<button @click="onCustomButtonClick">
			My custom button to select files
		</button>
		<h3>Custom button to open the file picker in "getFilesLink" mode</h3>
		<button @click="onCustomLinkButtonClick">
			My custom button to get public links
		</button>
		<h3>
			File picker component
		</h3>
		<p>
			NcWebdavFilePicker component is placed under the following line. It shows every button by default.
		</p>
		<hr>
		<NcWebdavFilePicker
			ref="filepicker"
			:commented-nc-oidc-config="oidcConfig"
			:commented-oidc-config-location="oidcConfigLocation"
			:nc-url="ncUrl"
			:nc-login="login"
			:nc-password="password"
			:nc-access-token="accessToken"
			:use-cookies="false"
			:multiple-upload="true"
			:multiple-download="true"
			:theme-color="color"
			:dark-mode="darkMode"
			:picker-is-open="false"
			:enable-get-files-path="true"
			:enable-get-files-link="true"
			:enable-download-files="true"
			:enable-get-save-file-path="true"
			:enable-get-upload-file-link="true"
			:enable-upload-files="true"
			:language="null"
			:use-webapppassword="true"
			@closed="onClosed"
			@manually-closed="onManuallyClosed"
			@filepicker-unauthorized="onUnauthorized"
			@files-downloaded="onFilesDownloaded"
			@files-uploaded="onFilesUploaded"
			@get-save-file-path="onGetSaveFilePath"
			@upload-path-link-generated="onUploadPathLinkGenerated"
			@get-files-link="onGetFilesLink"
			@get-files-path="onGetFilesPath">
			<template #get-files-link>
				<button style="background-color: #eeffee;">
					🔗 Custom button passed via a slot, get files links
				</button>
			</template>
		</NcWebdavFilePicker>
		<hr>
		<h3>Results</h3>
		<div v-for="(line, i) in resultLines" :key="i">
			<div v-if="line.path && line.url">
				{{ line.path }}
				<a :href="line.url">
					{{ line.url }}
				</a>
			</div>
			<div v-else>
				{{ line }}
			</div>
		</div>
		<div class="ribbon">
			<a href="https://github.com/eneiluj/nextcloud-webdav-filepicker" target="_blank">Repository and documentation</a>
		</div>
	</div>
</template>

<script>
import NcWebdavFilePicker from '../components/NcWebdavFilePicker'

export default {
	name: 'ExampleApp',

	components: {
		NcWebdavFilePicker,
	},

	props: [],

	data() {
		return {
			ncUrl: 'https://localhost/dev/server',
			login: '',
			password: '',
			accessToken: '',
			color: '#0082c9',
			darkMode: false,
			domainToAuthorize: window.location.protocol + '//' + window.location.host,
			resultLines: [],
			oidcConfigLocation: 'http://localhost/oidc-config.json',
			oidcConfig: {
				openIdConnect: {
					authority: 'http://localhost:8080/auth/realms/myrealm/.well-known/openid-configuration',
					client_id: 'superclient',
					client_secret: 'be4e2e3e-506e-4078-8919-5067edfa722a',
					popup_redirect_uri: 'http://localhost/oidc-callback.html',
					response_type: 'code',
					scope: 'openid email profile',
					automaticSilentRenew: true,
					accessTokenExpiringNotificationTime: 50,
				},
			},
		}
	},

	computed: {
	},

	watch: {
	},

	beforeMount() {
		const uri = window.location.search.substring(1)
		const params = new URLSearchParams(uri)
		const login = params.get('login')
		if (login) {
			this.login = login
		}
		const password = params.get('password')
		if (password) {
			this.password = password
		}
		const accessToken = params.get('accessToken')
		if (accessToken) {
			this.accessToken = accessToken
		}
		const url = params.get('url')
		if (url) {
			this.ncUrl = url
		}
		const color = params.get('color')
		if (color) {
			this.color = '#' + color
		}
		const darkMode = params.get('darkMode')
		if (darkMode === '1') {
			this.darkMode = true
		}
	},

	methods: {
		onCustomButtonClick() {
			this.$refs.filepicker.getFilesPath()
		},
		onCustomLinkButtonClick() {
			this.$refs.filepicker.getFilesLink({
				expirationDate: new Date('2050-01-01'),
				protectionPassword: 'example passwd',
				allowEdition: true,
				linkLabel: 'custom link label',
			})
		},
		onUnauthorized(detail) {
			console.debug('File picker failure, received unauthorized response code, check your credentials')
			console.debug('Response message: ')
			console.debug(detail.response)
		},
		onGetFilesPath(detail) {
			console.debug('something was selected')
			console.debug(detail)
			this.resultLines = ['- File paths:']
			detail.selection.forEach((l) => {
				this.resultLines.push(l)
			})
		},
		onGetFilesLink(detail) {
			console.debug('links were generated')
			console.debug(detail)
			this.resultLines = []
			if (detail.shareLinks) {
				this.resultLines.push('- Nextcloud public links:')
				detail.shareLinks.forEach((l) => {
					this.resultLines.push(l)
				})
			} else {
				this.resultLines.push('- Nextcloud public link creation failed. Link options are:')
				for (const k in detail.linkOptions) {
					this.resultLines.push(k + ': ' + detail.linkOptions[k])
				}
			}

			this.resultLines.push('- Path list:')
			detail.pathList.forEach((path) => {
				this.resultLines.push(path)
			})
			this.resultLines.push('- OCS URL:')
			this.resultLines.push(detail.ocsUrl)
			this.resultLines.push('- Share link template:')
			this.resultLines.push(detail.genericShareLink)
		},
		onGetSaveFilePath(detail) {
			console.debug('This target directory was selected')
			console.debug(detail)
			this.resultLines = ['This target directory was selected:', detail.path]
		},
		onUploadPathLinkGenerated(detail) {
			console.debug('This upload link was generated')
			console.debug(detail)
			this.resultLines = [`Upload link to ${detail.targetDir}:`, detail.link]
		},
		onFilesUploaded(detail) {
			console.debug('Files were uploaded')
			console.debug(detail.successFiles)
			console.debug(detail.errorFiles)
			this.resultLines = []
			if (detail.successFiles.length > 0) {
				this.resultLines.push(`- These files were uploaded in ${detail.targetDir}:`)
				detail.successFiles.forEach(file => {
					this.resultLines.push(file.name)
				})
			}
			if (detail.errorFiles.length > 0) {
				this.resultLines.push('!!! Those files could not be uploaded:')
				detail.errorFiles.forEach(file => {
					this.resultLines.push(file.name)
				})
			}
		},
		onFilesDownloaded(detail) {
			console.debug('something was downloaded')
			console.debug('failures')
			console.debug(detail.errorFilePaths)
			this.resultLines = ['- Downloaded files:']
			detail.successFiles.forEach(file => {
				console.debug('File : ' + file.name)
				console.debug(file)
				const reader = new FileReader()
				reader.readAsText(file)
				reader.onload = () => {
					console.debug(reader.result)
					this.resultLines.push('File ' + file.name + ': ' + reader.result.slice(0, 100) + '...')
				}
				reader.onerror = () => {
					console.error('Impossible to read downloaded file')
					console.debug(reader.error)
				}
			})
		},
		onClosed(e) {
			console.debug('file picker was closed')
		},
		onManuallyClosed(e) {
			console.debug('file picker was manually closed')
		},
	},
}
</script>

<style scoped lang="scss">
#example-app {
	width: 100%;
}

.ribbon {
	background-color: #0082c9;
	overflow: hidden;
	white-space: nowrap;
	position: fixed;
	right: -70px;
	top: 75px;
	-webkit-transform: rotate(45deg);
	-moz-transform: rotate(45deg);
	-ms-transform: rotate(45deg);
	-o-transform: rotate(45deg);
	transform: rotate(45deg);
	-webkit-box-shadow: 0 0 10px #888;
	-moz-box-shadow: 0 0 10px #888;
	box-shadow: 0 0 10px #888;

	a {
		border: 1px solid #faa;
		color: #fff;
		display: block;
		font: bold 100% 'Helvetica Neue', Helvetica, Arial, sans-serif;
		margin: 1px 0;
		padding: 10px 50px;
		text-align: center;
		text-decoration: none;
		text-shadow: 0 0 5px #444;
	}
}
</style>
