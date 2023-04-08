<template>
	<v-app>
		<v-container>
			<v-main>
				<h1>사용법</h1>
				<h3>사용자 등록 -> 테이블 컬럼 누를 시 내용 입력 -> 컬럼 클릭 시 상태변경(흰색: 일반, 녹색: 완료, 오렌지: 진행중, 핑크: 지연)</h3>
				<h5>(컬럼 모두 완료 처리 시 클리어 생성)</h5>
				<br />
				<member-save @reload="getAllData"></member-save>
				<br />
				<br />

				<todo-save
					ref="todoSaveRef"
					@reload="getAllData"
					:select-id="selectMemberId"
				></todo-save>
				<v-row>
					<v-col v-for="(item, index) in tableData" :key="index" :cols="computedCols" >
						<v-simple-table>
							<thead>
								<tr>
									<th class="th_header">
										{{ item.name }}
										<v-btn class="ml-auto" @click="deleteMember(item.id)"
											>삭제</v-btn
										>
									</th>
								</tr>
							</thead>
							<tbody>
								<tr v-for="i in 6" :key="i">
									<td
										v-if="item.todoList[i - 1]?.id"
										@click="updateStatus(item.todoList[i - 1])"
										:class="item.todoList[i - 1].status"
									>
										{{ item.todoList[i - 1].content }}
									</td>
									<td v-else-if="i === 6 && isAllCompleted(item.todoList)">
										<v-btn color="error" @click="deleteTodoList(item.id)"
											>clear</v-btn
										>
									</td>
									<td v-else-if="i === 6">
										<v-btn depressed disabled>clear</v-btn>
									</td>
									<td v-else @click="addTodoOpen(item.id)"></td>
								</tr>
							</tbody>
						</v-simple-table>
					</v-col>
				</v-row>
				<v-alert
					v-model="showAlert"
					type="error"
					@dismiss="showAlert = false"
					style="position: absolute; top: 60px; right: 0"
					transition="scale-transition"
				>
					{{ message }}
				</v-alert>
			</v-main>
		</v-container>
	</v-app>
</template>

<script>
import SaveMemberDialog from "@/components/SaveMemberDialog.vue";
import SaveTodoDialog from "@/components/SaveTodoDialog.vue";
export default {
	components: {
		"member-save": SaveMemberDialog,
		"todo-save": SaveTodoDialog,
	},
	name: "App",
	data() {
		return {
			selectMemberId: "",
			tableData: [],
			statusMap: {
				CREATE: "IN_PROGRESS", // 생성됨
				IN_PROGRESS: "COMPLETED", // 진행중
				COMPLETED: "DELAY", // 완료됨
				DELAY: "IN_PROGRESS", // 딜레이
			},
			showAlert: false,
			message: "",
		};
	},
	created() {
		this.getAllData();
	},
	computed: {
		computedCols() {
			return window.innerWidth >= 768 ? 3 : 12 / 2;
		},
		isAllCompleted() {
			return todoList => {
				if (todoList.length > 0) {
					return todoList.every(todoItem => todoItem.status === "COMPLETED");
				}
			};
		},
	},
	methods: {
		getAllData() {
			fetch("http://localhost:8080/api/member", {
				method: "GET",
			})
				.then(res => {
					if (res.ok) {
						return res.json();
					}
				})
				.then(data => {
					this.tableData = data;
				})
				.catch(err => {
					console.error(err);
				});
		},
		deleteMember(id) {
			fetch(`http://localhost:8080/api/member/${id}`, {
				method: "DELETE",
				headers: {
					"Content-Type": "application/x-www-form-urlencoded",
				},
			})
				.then(res => {
					if (res.ok) {
						console.log(res);
					} else {
						return res.text().then(v => {
							throw new Error(v);
						});
					}
				})
				.then(data => {
					this.message = `삭제가 완료 되었습니다.`;
					this.showAlert = true;
					setTimeout(() => {
						this.showAlert = false;
					}, 2000);
					console.log(data);
					this.getAllData();
				})
				.catch(err => {
					alert(err.message);
				});
		},
		updateStatus(id) {
			const params = new URLSearchParams();
			params.append("status", this.statusMap[id.status]);
			fetch(`http://localhost:8080/api/todo/update/${id.id}`, {
				method: "PATCH",
				headers: {
					"Content-Type": "application/x-www-form-urlencoded",
				},
				body: params,
			})
				.then(res => {
					this.getAllData();
					console.log(res);
				})
				.catch(err => {
					console.log(err);
				});
		},
		deleteTodoList(id) {
			fetch(`http://localhost:8080/api/todo/${id}`, {
				method: "DELETE",
				headers: {
					"Content-Type": "application/x-www-form-urlencoded",
				},
			})
				.then(res => {
					this.getAllData();
					console.log(res);
				})
				.catch(err => {
					console.log(err);
				});
		},
		addTodoOpen(memberId) {
			this.selectMemberId = memberId;
			this.$refs.todoSaveRef.todoDialog = true;
		},
	},
};
</script>
<style>
table {
	border: 1px solid lightgray;
}
.CREATE {
	background-color: white;
}

.IN_PROGRESS {
	background-color: #fec107;
}

.COMPLETED {
	background-color: #4caf50;
}

.DELAY {
	background-color: #fb9678;
}

.th_header {
	display: flex;
	align-items: center;
}
</style>
