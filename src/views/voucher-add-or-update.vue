<template>
	<el-dialog
		:title="!dataForm.id ? '新增' : '修改'"
		v-if="isAuth(['ROOT', 'VOUCHER:INSERT', 'VOUCHER:UPDATE'])"
		:close-on-click-modal="false"
		v-model="visible"
		width="500px"
		top="8vh"
	>
		<el-form :model="dataForm" ref="dataForm" :rules="dataRule" label-width="100px">
			<el-form-item label="代金券名称" prop="name">
				<el-input v-model="dataForm.name" size="medium" style="width:100%" clearable />
			</el-form-item>
			<el-form-item label="描述信息" prop="remark">
				<el-input
					v-model="dataForm.remark"
					size="medium"
					type="textarea"
					rows="3"
					style="width:100%"
					clearable
				/>
			</el-form-item>
			<el-form-item label="标签" prop="tag">
				<el-input v-model="dataForm.tag" size="medium" style="width:100%" clearable />
			</el-form-item>
			<el-form-item label="总数量" prop="totalQuota">
				<el-input-number v-model="dataForm.totalQuota" :min="0" :step="100" style="width:55%" size="small" />
				&nbsp;&nbsp;&nbsp;/&nbsp;张
			</el-form-item>
			<el-form-item label="代金券面额" prop="discount">
				<el-input-number v-model="dataForm.discount" :min="1" :max="50" style="width:55%" size="small" />
				&nbsp;&nbsp;&nbsp;/&nbsp;元
			</el-form-item>
			<el-form-item label="最低消费额" prop="withAmount">
				<el-input-number
					v-model="dataForm.withAmount"
					:min="0"
					:max="50"
					:step="10"
					style="width:55%"
					size="small"
				/>
				&nbsp;&nbsp;&nbsp;/&nbsp;元
			</el-form-item>
			<el-form-item label="赠送类型" prop="type">
				<el-select
					v-model="dataForm.type"
					class="input"
					placeholder="赠送类型"
					size="medium"
					clearable="clearable"
				>
					<el-option label="用户领取" value="1" />
					<el-option label="注册赠券" value="2" />
					<el-option label="补偿券" value="3" />
				</el-select>
			</el-form-item>
			<el-form-item label="领取限制" prop="limitQuota">
				<el-select
					v-model="dataForm.limitQuota"
					class="input"
					placeholder="领取限制"
					size="medium"
					clearable="clearable"
				>
					<el-option label="无限制" value="0" />
					<el-option label="有限制" value="1" />
				</el-select>
			</el-form-item>
			<el-form-item label="有效期限" prop="timeType">
				<el-select
					v-model="dataForm.timeType"
					class="input"
					placeholder="有效期限"
					size="medium"
					clearable="clearable"
					@change="changeHandle"
					@clear="clearHandle"
				>
					<el-option label="无期限" value="0" />
					<el-option label="有效天数" value="1" />
					<el-option label="有效日期" value="2" />
				</el-select>
			</el-form-item>
			<el-form-item label="有效天数" prop="days" v-if="daysVisible">
				<el-input-number v-model="dataForm.days" :min="1" style="width:55%" size="small" />
			</el-form-item>
			<el-form-item label="有效日期" prop="date" v-if="dateVisible">
				<el-date-picker
					v-model="dataForm.date"
					type="daterange"
					range-separator="~"
					start-placeholder="开始日期"
					end-placeholder="结束日期"
					size="medium"
				></el-date-picker>
			</el-form-item>
		</el-form>
		<template #footer>
			<span class="dialog-footer">
				<el-button size="medium" @click="visible = false">取消</el-button>
				<el-button type="primary" size="medium" @click="dataFormSubmit">确定</el-button>
			</span>
		</template>
	</el-dialog>
</template>

<script>
export default {
	data: function() {
		return {
			visible: false,
			dataForm: {
				name: null,
				remark: null,
				tag: null,
				totalQuota: 0,
				discount: null,
				withAmount: null,
				type: null,
				limitQuota: null,
				timeType: null,
				days: null,
				date: []
			},
			dataRule: {
				name: [{ required: true, message: '名称不能为空' }],
				type: [{ required: true, message: '赠送类型不能为空' }],
				limitQuota: [{ required: true, message: '领取限制不能为空' }],
				timeType: [{ required: true, message: '有效期限不能为空' }],
				date: [{ validator: this.validateDate }]
			},
			daysVisible: false,
			dateVisible: false
		};
	},

	methods: {
		validateDate: function(rule, value, callback) {
			if (value == null) {
				callback(new Error('日期范围不能为空'));
			} else {
				callback();
			}
		},
		changeHandle: function(e) {
			this.dataForm.days = null;
			this.dataForm.date = null;
			if (e == '0') {
				this.daysVisible = false;
				this.dateVisible = false;
			} else if (e == '1') {
				this.daysVisible = true;
				this.dateVisible = false;
			} else if (e == '2') {
				this.daysVisible = false;
				this.dateVisible = true;
			}
		},
		clearHandle: function() {
			this.dataForm.days = null;
			this.dataForm.date = null;
			this.daysVisible = false;
			this.dateVisible = false;
		},
		init: function() {
			let that = this;
			this.daysVisible = false;
			this.dateVisible = false;
			that.visible = true;
			that.$nextTick(() => {
				that.$refs['dataForm'].resetFields();
			});
		},
		dataFormSubmit: function() {
			let that = this;
			this.$refs['dataForm'].validate(valid => {
				if (valid) {
					let data = {
						name: that.dataForm.name == '' ? null : that.dataForm.name,
						remark: that.dataForm.remark == '' ? null : that.dataForm.remark,
						tag: that.dataForm.tag == '' ? null : that.dataForm.tag,
						totalQuota: that.dataForm.totalQuota,
						discount: that.dataForm.discount,
						withAmount: that.dataForm.withAmount,
						type: that.dataForm.type == '' ? null : that.dataForm.type,
						limitQuota: that.dataForm.limitQuota,
						timeType: that.dataForm.timeType == 0 ? null : that.dataForm.timeType,
						days: that.dataForm.days
					};
					if (that.dataForm.date != null && that.dataForm.date.length == 2) {
						let startTime = that.dataForm.date[0];
						let endTime = that.dataForm.date[1];
						data.startTime = dayjs(startTime).format('YYYY-MM-DD');
						//有效期截止到次日零点，所以日期要往后偏移1天
						data.endTime = dayjs(endTime)
							.add(1, 'day')
							.format('YYYY-MM-DD');
					}
					that.$http('voucher/insertVoucher', 'POST', data, true, function(resp) {
						if (resp.rows == 1) {
							that.$message({
								message: '操作成功',
								type: 'success',
								duration: 1200
							});
							that.visible = false;
							that.$emit('refreshDataList');
						} else {
							that.$message({
								message: '操作失败',
								type: 'error',
								duration: 1200
							});
						}
					});
				}
			});
		}
	}
};
</script>

<style lang="less" scoped="scoped"></style>
