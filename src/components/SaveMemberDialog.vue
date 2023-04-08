<template>
	<div>
		<v-dialog v-model="memberDialog" width="500">
			<template v-slot:activator="{ on, attrs }">
				<v-btn color="red lighten-2" dark v-bind="attrs" v-on="on">
					사용자등록
				</v-btn>
			</template>

			<v-card>
				<v-card-title class="text-h5 grey lighten-2">
					사용자를 등록해주세요.
				</v-card-title>
				<br />
				<br />
				<v-row style="width: 100%">
					<v-col cols="1"></v-col>
					<v-text-field
						v-model="name"
						label="이름"
						clearable
						@keyup.enter="saveMember()"
					></v-text-field>
					<v-col cols="1"></v-col>
				</v-row>
				<br />
				<br />
				<v-divider></v-divider>

				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn color="primary" text @click="saveMember()"> 등록 </v-btn>
					<v-btn color="primary" text @click="cancel()"> 취소 </v-btn>
				</v-card-actions>
			</v-card>
		</v-dialog>
		<v-alert
			v-model="showAlert"
			type="success"
			@dismiss="showAlert = false"
			style="position: absolute; top: 60px; right: 0"
			transition="scale-transition"
		>
			{{ message }}
		</v-alert>
	</div>
</template>

<script>
export default {
	data() {
		return {
			memberDialog: false,
			name: "",
			showAlert: false,
			message: "",
		};
	},
	methods: {
		saveMember() {
			if(!this.name) {
				alert('이름을 입력해 주세요')
				return;
			}

			const params = new URLSearchParams();
			params.append("name", this.name);
			fetch("http://localhost:8080/api/member/save", {
				method: "POST",
				headers: {
					"Content-Type": "application/x-www-form-urlencoded",
				},
				body: params,
			})
				.then(res => {
					if (res.ok) {
						return res.json();
					} else {
						return res.text().then(v => {
							throw new Error(v);
						});
					}
				})
				.then(data => {
					this.message = data.name + "님 등록이 완료되었습니다.";
					this.showAlert = true;
					setTimeout(() => {
						this.showAlert = false;
					}, 2000);
					this.$emit("reload");
					this.cancel();
				})
				.catch(err => {
					alert(err.message);
				});
		},
		test() {
			alert("d");
		},
		cancel() {
			this.name = "";
			this.memberDialog = false;
		},
	},
};
</script>
