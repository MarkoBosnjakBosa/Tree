<template>
	<div id="tree">
		<tree-buttons @createpage="createPage" @createsection="createSection" @createelement="createElement" @removeelement="removeElement"/>
		<tree-structure :tree="tree" @makeactive="activate" @dropelement="onDropElement"/>
	</div>
</template>

<script>
import TreeButtons from "@/components/TreeButtons.vue";
import TreeStructure from "@/components/TreeStructure.vue";

export default {
	name: "App",
	components: {
		TreeButtons,
		TreeStructure,
	},
	data() {
		return {
			treeElements: [],
			tree: []
		}
	},
	methods: {
		createPage(type) {
			var id = Math.floor(Math.random() * 1000);
			var pages = Number(this.treeElements.filter(treeElement => treeElement.parentid == 0).length);
			this.treeElements.filter(treeElement => treeElement.active == "active").map(treeElement => treeElement.active = "");
			this.treeElements.push({"id" : id, "parentid" : 0, "type" : type, "orderid" : pages, "active" : "active"});
			this.createTree();
		},
		createSection(type) {
			if(this.treeElements.length > 0) {
				var id = Math.floor(Math.random() * 1000);
				var selectedElement = document.querySelector(".active");
				if(selectedElement && selectedElement.classList.contains("page")) {
					var parentId = selectedElement.getAttribute("data-fieldid");
					var pages = Number(this.treeElements.filter(treeElement => treeElement.parentid == parentId).length);
					this.treeElements.push({"id" : id, "parentid" : Number(parentId), "type" : type, "orderid" : pages, "active" : ""});
					this.createTree();
				}
			}
		},
		createElement(type) {
			if(this.treeElements.length > 0) {
				var id = Math.floor(Math.random() * 1000);
				var selectedElement = document.querySelector(".active");
				if(selectedElement && (selectedElement.classList.contains("page") || selectedElement.classList.contains("section"))) {
					var parentId = selectedElement.getAttribute("data-fieldid");
					var pages = Number(this.treeElements.filter(treeElement => treeElement.parentid == parentId).length);
					this.treeElements.push({"id" : id, "parentid" : Number(parentId), "type" : type, "orderid" : pages, "active" : ""});
					this.createTree();
				}
			}
		},
		activate(selectedElementId) {
			this.treeElements.filter(treeElement => treeElement.active == "active").map(treeElement => treeElement.active = "");
			this.treeElements.filter(treeElement => treeElement.id == selectedElementId).map(treeElement => treeElement.active = "active");
			this.createTree();
		},
		createTree() {
			var temporaryTree = [];
			var mappedArray = {};
			var arrayElem;
			var mappedElem;
			for(var i = 0; i < this.treeElements.length; i++) {
				arrayElem = this.treeElements[i];
				mappedArray[arrayElem.id] = arrayElem;
				mappedArray[arrayElem.id]["children"] = [];
			}
			for(var id in mappedArray) {
				if(Object.prototype.hasOwnProperty.call(mappedArray, id)) {
					mappedElem = mappedArray[id];
					if(mappedElem.parentid) {
						mappedArray[mappedElem["parentid"]]["children"].push(mappedElem);
						mappedArray[mappedElem["parentid"]]["children"].sort((x, y) => parseFloat(x.orderid) - parseFloat(y.orderid));
					} else {
						temporaryTree.push(mappedElem);
						temporaryTree.sort((x, y) => parseFloat(x.orderid) - parseFloat(y.orderid));
					}
				}
			}
			this.tree = temporaryTree;
		},
		removeElement() {
			var selectedElement = document.querySelector(".active");
			var shouldRemove = false;
			if(selectedElement) {
				if(selectedElement.children.length == 1) {
					shouldRemove = true;
				}
				else if(selectedElement.children.length > 1) {
					if(selectedElement.children[1].querySelector("ul > li").classList.contains("emptyPage") || selectedElement.children[1].querySelector("ul > li").classList.contains("emptySection")) {
						shouldRemove = true;
					}
				}
				if(shouldRemove == true) {
					var selectedId = Number(selectedElement.getAttribute("data-fieldid"));
					var index = this.treeElements.map(treeElement => treeElement.id).indexOf(selectedId);
					this.treeElements.splice(index, 1);
					this.createTree();
				}
			}
		},
		onDropElement(draggedElement, parentUl) {
			this.treeElements.filter(treeElement => treeElement.id == draggedElement).map(treeElement => treeElement.parentid = parentUl);
			this.treeElements.filter(treeElement => treeElement.parentid == parentUl).map(treeElement => {
				var el = document.getElementById("element_" + treeElement.id);
				var orderId = ([...el.parentElement.children].indexOf(el));
				treeElement.orderid = orderId;
			});
			this.createTree();
		}
	}
}
</script>
