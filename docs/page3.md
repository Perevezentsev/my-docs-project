```
 // Восстановление ветки 
MATCH (branch:Branch {id:$branchId}) - [:INSTANCE_PROXY] -> (branchProxy:OntologyInstance)
REMOVE branch.removed
REMOVE branchProxy.removed

```

```
 // Восстановление bpmn и элементов 
MATCH (branch:Branch {id:$branchId}) 
OPTIONAL MATCH (branch) <- [:IN_BRANCH] - (bpmn:BPMN)
OPTIONAL MATCH (bpmn) - [:INSTANCE_PROXY] -> (bpmnProxy:OntologyInstance) 
OPTIONAL MATCH (bpmn) <- [:ELEMENT_OF] - (bpmnElement:BPMNElement) 
  WHERE bpmn.dateUpdated = branch.dateUpdated AND bpmnElement.dateUpdated = branch.dateUpdated 
REMOVE bpmn.removed
REMOVE bpmnProxy.removed
REMOVE bpmnElement.removed

```

```
 // Восстановление dmn и элементов 
MATCH (branch:Branch {id:$branchId}) 
OPTIONAL MATCH (branch) <- [:IN_BRANCH] -  (table:DecisionTable) <- [:IN_TABLE] - (tableElement)
  WHERE table.dateUpdated = branch.dateUpdated AND tableElement.dateUpdated = branch.dateUpdated 
REMOVE table.removed
REMOVE tableElement.removed

```

```
 // Восстановление макетов
MATCH (branch:Branch {id:$branchId}) 
OPTIONAL MATCH (branch) <- [:IN_BRANCH] - (frame:Frame)
  WHERE frame.dateUpdated = branch.dateUpdated 
REMOVE frame.removed

```

```
 // Восстановление коллекций
MATCH (branch:Branch {id:$branchId}) 
 OPTIONAL MATCH (branch) <- [:IN_BRANCH] - (collection:OntologyClassCollection) - [h:HAS_STATE] -> (state)
  WHERE h.to IS NULL AND state.dateUpdated = branch.dateUpdated 
REMOVE state.removed

```

```
 // Восстановление онтологий
MATCH (branch:Branch {id:$branchId}) 
 OPTIONAL MATCH (branch) <- [:IN_BRANCH] - (ontology:Ontology) - [h:HAS_STATE] -> (state)
  WHERE h.to IS NULL AND state.dateUpdated = branch.dateUpdated 
REMOVE state.removed
WITH DISTINCT branch, ontology
OPTIONAL MATCH (ontology) <- [:IN_ONTOLOGY] - (toOntology:ClassToOntologyRelation) - [h:HAS_STATE] -> (state)
  WHERE h.to IS NULL AND state.dateUpdated = branch.dateUpdated 
REMOVE state.removed  
WITH DISTINCT branch, toOntology
OPTIONAL MATCH (toOntology) <- [:IN_ONTOLOGY] - (class:OntologyClass) - [h:HAS_STATE] -> (state)
 WHERE h.to IS NULL AND NOT EXISTS {MATCH (class) - [:IN_ONTOLOGY] -> (otherToOntology:ClassToOntologyRelation)
              WHERE ID(toOntology) <> ID(otherToOntology) }
REMOVE state.removed   
WITH DISTINCT branch, class
OPTIONAL MATCH (class) <- [:IN_CLASS] - (attr:OntologyClassAttribute) - [h:HAS_STATE] -> (state)
  WHERE h.to IS NULL AND state.dateUpdated = branch.dateUpdated 
REMOVE state.removed   
WITH DISTINCT branch, class
OPTIONAL MATCH (class) <- [:CLASS_RELATION] - (relation:OntologyClassRelation)  - [h:HAS_STATE] -> (state)
              WHERE h.to IS NULL AND state.dateUpdated = branch.dateUpdated 
REMOVE state.removed 
WITH DISTINCT branch, class
 OPTIONAL MATCH (class) <- [:INSTANCE_OF] - (classInstance:OntologyInstance)
REMOVE classInstance.removed 
```