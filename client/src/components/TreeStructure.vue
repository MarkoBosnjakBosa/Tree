<template>
	<ul id="treeStructure">
		<li v-for="branch in tree" :key="branch.id" :id="'element_' + branch.id" :data-fieldid="branch.id" :orderid="branch.orderid" draggable="true" :class="'bfForm ' + branch.type + ' ' + branch.active" @dragstart="onDragStart($event)" @dragover.prevent="onDragOver($event)" @dragleave.prevent="onDragLeave($event)" @drop.prevent="onDrop($event)">
			<span @click="makeActive(branch.id)"> {{ branch.type }} - {{ branch.id }}</span>
			<ul :id="'ul_' + branch.id" :data-type="branch.type">
				<li v-if="branch.children.length < 1" class="emptyPage"></li>
				<li v-for="majorChild in branch.children" :key="majorChild.id" :id="'element_' + majorChild.id" :data-fieldid="majorChild.id" :orderid="majorChild.orderid" draggable="true" :class="'bfForm ' + majorChild.type + ' ' + majorChild.active" @dragstart="onDragStart($event)" @dragover.prevent="onDragOver($event)" @dragleave.prevent="onDragLeave($event)" @drop.prevent="onDrop($event)">
					<span @click="makeActive(majorChild.id)">{{ majorChild.type }} - {{ majorChild.id }}</span>
					<ul v-if="majorChild.type == 'section'" :id="'ul_' + majorChild.id" :data-type="majorChild.type">
						<li v-if="majorChild.children.length < 1" class="emptySection"></li>
						<li v-for="minorChild in majorChild.children" :key="minorChild.id" :id="'element_' + minorChild.id" :data-fieldid="minorChild.id" :orderid="minorChild.orderid" draggable="true" :class="'bfForm ' + minorChild.type + ' ' + minorChild.active" @dragstart="onDragStart($event)" @dragover.prevent="onDragOver($event)" @dragleave.prevent="onDragLeave($event)" @drop.prevent="onDrop($event)">
							<span @click="makeActive(minorChild.id)">{{ minorChild.type }} - {{ minorChild.id }}</span>
						</li>
					</ul>
				</li>
			</ul>
		</li>
	</ul>
</template>

<script>
export default {
	name: "tree-structure",
	data() {
		return {
			dragging: null
		}
	},
	props: {
		tree: Array
	},
	methods: {
		makeActive(clickedId) {
			this.$emit("makeactive", clickedId);
		},
		onDragStart(event) {
			this.dragging = this.getTarget(event.target);
			event.dataTransfer.setData("id", event.target.getAttribute("data-fieldid"));
			var elementClass = "";
			if(event.target.classList.contains("page")) elementClass = "page";
			if(event.target.classList.contains("section")) elementClass = "section";
			event.dataTransfer.setData("class", elementClass);
		},
		onDragOver(event) {
			var target = this.getTarget(event.target);
			var bounding = target.getBoundingClientRect();
			var offset = bounding.y + (bounding.height/2);
			var difference = event.clientY - offset;
			if(difference > 0) {
				target.style["border-bottom"] = "3px solid blue";
				target.style["border-top"] = "";
			} else {
				target.style["border-top"] = "3px solid blue";
				target.style["border-bottom"] = "";
			}
		},
		onDragLeave(event) {
			var target = this.getTarget(event.target);
			target.style["border-bottom"] = "";
			target.style["border-top"] = "";
		},
		onDrop(event) {
			event.stopImmediatePropagation();
			var draggedElement = event.dataTransfer.getData("id");
			var draggedElementClass = event.dataTransfer.getData("class");
			var target = this.getTarget(event.target);
			if(((draggedElementClass != "page" || !target.classList.contains("emptyPage")) && (draggedElementClass != "page" || target.parentNode.getAttribute("data-type") != "page")) && ((draggedElementClass != "page" || !target.classList.contains("emptySection")) && (draggedElementClass != "page" || target.parentNode.getAttribute("data-type") != "section")) && ((draggedElementClass != "section" || !target.classList.contains("emptySection")) && (draggedElementClass != "section" || target.parentNode.getAttribute("data-type") != "section"))) {
				var parentUl = Number(target.parentNode.id.split("_")[1]) || 0;
				if(parentUl == 0) {
					if(target.classList.contains("page") && draggedElementClass != "page") {
						parentUl = Number(target.getAttribute("data-fieldid"));
					}
				}
				if(target.style["border-bottom"] !== "") {
					target.style["border-bottom"] = "";
					target.parentNode.insertBefore(this.dragging, target.nextSibling);
				} else {
					try {
						target.style["border-top"] = "";
						if(target.parentNode.getAttribute("id") != "treeStructure") {
							target.parentNode.insertBefore(this.dragging, target);
						}
						if(target.parentNode.getAttribute("id") == "treeStructure" && draggedElementClass == "page") {
							target.parentNode.insertBefore(this.dragging, target);
						}
					} catch(error) { console.log(""); }
				}
				this.$emit("dropelement", draggedElement, parentUl);
			} else {
				target.style["border-top"] = "";
				target.style["border-bottom"] = "";
			}
		},
		getTarget(target) {
			while(target.nodeName.toLowerCase() != "li" && target.nodeName.toLowerCase() != "body") {
				target = target.parentNode;
			}
			if(target.nodeName.toLowerCase() == "body'") {
				return false;
			} else {
				return target;
			}
		}
	},
}
</script>

<style scoped>
	ul {
		padding-left: 0px;
		list-style-type: none;
		margin: 0px;
		padding: 0px;
	}
	li {
		cursor: move;
		display: block;
		padding: 15px;
		background: #fff;
		outline: 1px dashed #000;
	}
	li.active > span {
		color: red;
	}
	.page {
		border-bottom: 0px !important;
	}
	.section {
		margin-left: 20px;
	}
	.page .element {
		margin-left: 20px;
	}
	.section .element {
		margin-left: 40px;
	}
	.emptyPage {
		padding: 5px;
		border-bottom: 0px !important;
		margin-left: 20px;
	}
	.emptySection {
		padding: 5px;
		border-bottom: 0px !important;
		margin-left: 40px;
	}
	span {
		text-transform: uppercase;
	}
</style>
