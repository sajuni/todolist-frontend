<template>
	<v-app>
		<v-main>
			<h1>사용자를 먼저 등록해 주세요.</h1>
			<v-dialog v-model="dialog" width="500">
				<template v-slot:activator="{ on, attrs }">
					<v-btn color="red lighten-2" dark v-bind="attrs" v-on="on">
						사용자등록
					</v-btn>
				</template>

				<v-card>
					<v-card-title class="text-h5 grey lighten-2">
						사용자를 등록해주세요
					</v-card-title>
					<v-text-field v-model="name" label="이름" clearable></v-text-field>
					<v-divider></v-divider>

					<v-card-actions>
						<v-spacer></v-spacer>
						<v-btn color="primary" text @click="save()"> 등록 </v-btn>
						<v-btn color="primary" text @click="cancel()"> 취소 </v-btn>
					</v-card-actions>
				</v-card>
			</v-dialog>
		</v-main>
	</v-app>
</template>

<script>
export default {
	name: "App",
	data() {
		return {
			dialog: false,
			name: "",
		};
	},
	created() {
		fetch("http://localhost:8080/api/member/12", {
			method: "GET",
		})
			.then(res => {
				console.log(res);
			})
			.catch(err => {
				console.error(err);
			});
	},
	methods: {
		save() {
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
					return res.json();
				})
				.then(data => {
					alert(data.name + "님 등록이 완료되었습니다.");
					this.cancel();
				})
				.catch(err => {
					console.error(err);
				});
		},
		cancel() {
			this.dialog = false;
			this.name = "";
		},
	},
};
</script>
