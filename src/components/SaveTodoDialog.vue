<template>
	<div>
		<v-dialog v-model="todoDialog" width="500">
			<v-card>
				<v-card-title class="text-h5 grey lighten-2">
					할일을 등록해주세요.
				</v-card-title>
				<br />
				<br />
				<v-row style="width: 100%">
					<v-col cols="1"></v-col>
					<v-col cols="10">
						<v-text-field
							v-model="content"
							label="컨텐츠이름"
							clearable
							@keyup.enter="saveTodo()"
						></v-text-field>
					</v-col>
					<v-col></v-col>
				</v-row>
				<br />
				<br />
				<v-divider></v-divider>

				<v-card-actions>
					<v-spacer></v-spacer>
					<v-btn color="primary" text @click="saveTodo()"> 등록 </v-btn>
					<v-btn color="primary" text @click="cancel()"> 취소 </v-btn>
				</v-card-actions>
			</v-card>
		</v-dialog>
		<v-alert
			v-model="showAlert"
			type="info"
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
	props: {
		selectId: {},
	},
	data() {
		return {
			todoDialog: false,
			content: "",
			selectMemberId: "",
			showAlert: false,
			message: "",
		};
	},
	methods: {
		saveTodo() {
			const params = new URLSearchParams();
			params.append("memberId", this.selectId);
			params.append("content", this.content);
			fetch("http://localhost:8080/api/todo/save", {
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
					this.message = `${data.content} 등록이 완료 되었습니다.`;
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
		cancel() {
			this.todoDialog = false;
			this.content = "";
		},
	},
};
</script>
