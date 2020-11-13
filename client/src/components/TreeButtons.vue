<template>
	<div class="buttonsDiv">
		<button type="button" id="newPage" class="addButton" data-type="page" @click="createPage">Page</button>
		<button type="button" id="newSection" class="addButton" data-type="section" @click="createSection">Section</button>
		<button type="button" id="newElement" class="addButton" data-type="element" @click="createElement">Element</button>
		<button type="button" id="remove" class="removeButton" data-type="remove" @click="removeElement" @mouseover="allowRemoving">Remove</button>
	</div>
</template>

<script>
export default {
	name: "tree-buttons",
	methods: {
		createPage() {
			this.$emit("createpage", "page");
		},
		createSection() {
			this.$emit("createsection", "section");
		},
		createElement() {
			this.$emit("createelement", "element");
		},
		removeElement() {
			this.$emit("removeelement");
		},
		allowRemoving() {
			var selectedElement = document.querySelector(".active");
			if(selectedElement){
				if(selectedElement.children.length == 1) {
					document.getElementById("remove").classList.remove("disabled");
				}
				else if(selectedElement.children.length > 1) {
					if(selectedElement.children[1].querySelector("ul > li").classList.contains("emptyPage") || selectedElement.children[1].querySelector("ul > li").classList.contains("emptySection")) {
						document.getElementById("remove").classList.remove("disabled");
					} else {
						document.getElementById("remove").classList.add("disabled");
					}
				} else {
					document.getElementById("remove").classList.add("disabled");
				}
			} else {
				document.getElementById("remove").classList.add("disabled");
			}
		}
	}
}
</script>

<style scoped>
	.buttonsDiv {
		margin-bottom: 20px;
	}
	.addButton {
		border: 2px solid #000;
		border-radius: 5px;
		background-color: #0d8bf2;
		color: #fff;
		padding: 14px 28px;
		margin-right: 10px;
		font-size: 16px;
		cursor: pointer;
	}
	.removeButton {
		border: 2px solid #000;
		border-radius: 5px;
		background-color: #ff0000;
		color: #fff;
		padding: 14px 28px;
		margin-right: 10px;
		font-size: 16px;
		cursor: pointer;
	}
	#remove.disabled {
		cursor: not-allowed;
	}
</style>
