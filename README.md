# mteqa
Multi-Target Embodied Question Answering


## Citation

    @inproceedings{yu2019mteqa,
      title={Multi-Target Embodied Question Answering},
      author={Yu, Licheng and Chen, Xinlei and Gkioxari, Georgia and Bansal, Mohit and Berg, Tamara L and Batra, Dhruv},
      booktitle={CVPR},
      year={2019}
    }


## Data Generation

Go to `eqa_data` folder and do the followings:

### How to generate question-answer pairs
1. Run `./scripts/run_question_generation.sh`, this would save qas to `cache/question-gen-outputs/questions_pruned_v2.json`.
2. Run `python question-gen/make_object_id_to_cat.py` to make mapping from hid.oid to fine_class.

### Generate graphs, connMaps, and shortest-paths
1. Run `python shortest-path-gen/generate-conn-maps.py` to build graph and connMaps. This would:
    - save graphs to `cache/3d-graphs`.
    - save connMaps to `cache/target-obj-conn-maps`.
    - update qas to `cache/question-gen-outputs/questions_pruned_mt_with_conn.json`.
2. Run `python shortest-path-gen/get_best_view.py` saving best-view points and ious. Note the points information [x, 1.0, z, yaw] is in coordinate system (not grid system). 
3. Run `python shortest-path-gen/add_program_mt.py` to add parsed programs to questions, update qas to `cache/question-gen-outputs/questions_pruned_mt_with_conn_program.json`.
4. Run `python shortest-path-gen/generate-bestview-paths-mt.py` to compute shortest paths connecting start, best-view, end points. Note we intentionally make a faked end point for testing if agent learns to stop at right position (before the faked end point).
5. Run `python shortest-path-gen/filter_questions_mt.py` to filter out those questions without shortest paths or of low entropy, saving filtered questions to `questions_mt_paths_nearby_source_best_view_program.json`.

### For installing House3D
Follow the instructions[https://github.com/facebookresearch/House3D].


## Imitation Learning for Nav+Ctrl+cVQA

Go to `nav_loc_vqa` folder and do the followings:

### Prepare House Data (conn-maps, graphs, shortest-paths, images, features, etc)
1. Copy or Symlink `../eqa_data/cache/question-gen-outputs`, `../eqa_data/cache/shortest-paths-mt`, `../eqa_data/target-obj-bestview-pos` and `../eqa_data/target-obj-conn-maps` to `./data`
2. Run `./tools/generate_path_imgs.py` and `./tools/generate_path_cube_imgs.py` to extract 1st-person images along paths.
3. Run `./tools/extract_feats.py` and `./tools/extract_cube_feats.py` to extract 1st-person features along paths.
4. Run `./tools/compute_meta_info.py` to compute room meta info.

### Train and Eval IL
1. Run `./tools/prepro_imitation_data.py` and `./tools/prepro_reinforce_data.py`
2. Run `./tools/eval_gd_path.py` to get results on ground-truth paths.

### Evaluate RL-finetuned Model (after checking eqa_nav)
1. Run `./tools/evaleqa_nr.py`


## Reinforcement Learning Finetuning for Navigators

Go to `eqa_nav` folder and do the followings:

### Prepare House Data (conn-maps, graphs, shortest-paths, images, features, etc)
1. Copy or Symlink `../eqa_data/cache/question-gen-outputs`, `../eqa_data/cache/shortest-paths-mt`, `../eqa_data/target-obj-bestview-pos` and `../eqa_data/target-obj-conn-maps` to `./data`
2. Copy or Symlink `../nav_loc_eqa/cache/path_feats`, `../nav_loc_eqa/cache/path_images` and `../nav_loc_eqa/cache/path_to_room_meta` to `./cache`.

### Prepare Navigation Data
1. Run `./tools/prepro_im.py` for imitation learning data.
2. Run `./tools/prepro_rl.py` for reinforcement finetuning data.

### Train IL-based room-navigator and object-navigator
1. Run `./tools/train_im_object.py` and `./tools/train_im_room.py` for imitation learning.
2. Run `./tools/eval_im_object.py` and `./tools/eval_im_room.py` for imitation learning evaluation.

### Finetune using RL
1. Run `./tools/train_rl_object.py` and `./tools/train_rl_room.py` for reinforcement finetuning.
2. Run `./tools/eval_nav_object.py` and `./tools/eval_nav_room.py` for navigation evaluation.




