
# 模版数据表结构：
```mysql
CREATE TABLE `sys_org` (
  `id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '组织id',
  `parent_id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '父级组织id',
  `name` varchar(50) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '组织名称',
  `type` tinyint NOT NULL COMMENT '组织类型(字典：ORG_TYPE,0:组织，1:部门，2:团队)',
  `code` varchar(20) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci NOT NULL COMMENT '组织编码',
  `sort_index` int DEFAULT NULL COMMENT '序号',
  `create_time` timestamp NULL DEFAULT NULL COMMENT '创建时间',
  `update_time` timestamp NULL DEFAULT NULL COMMENT '更新时间',
  `create_user_id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '创建者',
  `update_user_id` varchar(64) CHARACTER SET utf8mb4 COLLATE utf8mb4_general_ci DEFAULT NULL COMMENT '更新者',
  `is_deleted` tinyint DEFAULT '0' COMMENT '逻辑删除 1:是，0:否',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_general_ci COMMENT='系统组织架构表';
```

# 代码模板  
# AbstractEntity

```java
package com.crunii.common.core.entity;

import com.baomidou.mybatisplus.annotation.FieldFill;
import com.baomidou.mybatisplus.annotation.TableField;
import com.baomidou.mybatisplus.annotation.TableLogic;
import com.baomidou.mybatisplus.extension.activerecord.Model;
import com.fasterxml.jackson.annotation.JsonIgnore;
import lombok.Data;

import java.time.LocalDateTime;

/**
 * 实体抽象类
 *
 * @Author Liuyh
 * @Date 2023/3/12
 * @Version V0.0.1
 */
@Data
public abstract class AbstractEntity<T extends AbstractEntity<?>> extends Model<T> {

    /**
     * 主键
     */
    private String id;

    /**
     * 是否删除(0:否，1:是)
     */
    @TableLogic
    @TableField("is_deleted")
    @JsonIgnore
    private Boolean isDeleted;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime createTime;

    /**
     * 更新时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime updateTime;

    /**
     * 创建人id
     */
    @TableField(fill = FieldFill.INSERT)
    private String createUserId;

    /**
     * 更新人id
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private String updateUserId;

}
```

# Entity

```java
package com.crunii.itss.entity.po.sys;

import com.crunii.common.core.entity.AbstractEntity;
import com.crunii.common.mybatis.annotation.SortCondition;
import com.crunii.common.mybatis.annotation.SortIndex;
import lombok.Data;

/**
 * 组织Entity
 *
 * @Author Liuyh
 * @Date 2023/3/12
 * @Version V0.0.1
 */
@Data
public class SysOrg extends AbstractEntity<SysOrg> {

    /**
     * 父级组织id
     */
    @SortCondition
    private String parentId;

    /**
     * 组织名称
     */
    private String name;

    /**
     * 组织类型(字典：DEPT_TYPE,0:组织，1:部门，2:团队)
     */
    private Integer type;

    /**
     * 组织编码
     */
    private String code;

    /**
     * 序号
     */
    @SortIndex
    private Integer sortIndex;
}


```

## Mapper

```java
package com.crunii.itss.mapper.sys;

import com.baomidou.mybatisplus.core.mapper.BaseMapper;
import com.crunii.itss.entity.po.sys.SysOrg;
import org.apache.ibatis.annotations.Mapper;

/**
 * 组织Mapper
 *
 * @Author Liuyh
 * @Date 2023/3/12
 * @Version V0.0.1
 */
@Mapper
public interface SysOrgMapper extends BaseMapper<SysOrg> {
}
```

## IService

```java
package com.crunii.itss.service.sys;

import com.crunii.common.core.entity.dto.BasicIdListDTO;
import com.crunii.common.mybatis.query.Page;
import com.crunii.common.mybatis.service.ICruniiService;
import com.crunii.itss.entity.dto.sys.SysOrgPageDTO;
import com.crunii.itss.entity.dto.sys.SysOrgSaveDTO;
import com.crunii.itss.entity.po.sys.SysOrg;
import com.crunii.itss.entity.vo.sys.SysOrgPageVO;
import com.crunii.itss.entity.vo.sys.SysOrgTreeVO;

import java.util.List;

/**
 * 组织Service
 *
 * @Author Liuyh
 * @Date 2023/3/13
 * @Version V0.0.1
 */
public interface ISysOrgService extends ICruniiService<SysOrg> {
    /**
     * 新增/修改组织
     *
     * @param saveDTO 组织信息 {@link SysOrgSaveDTO}
     * @return 组织信息
     */
    Boolean save(SysOrgSaveDTO saveDTO);

    /**
     * 删除组织
     *
     * @param id 组织id
     */
    void deleteSysOrg(String id);

    /**
     * 批量删除组织
     *
     * @param idListDTO 组织id集合 {@link BasicIdListDTO}
     */
    void batchDeleteSysOrg(BasicIdListDTO idListDTO);

    /**
     * 查询组织树
     *
     * @return 系统架构树
     */
    List<SysOrgTreeVO> getSysOrgTree();

    /**
     * 分页条件查询组织列表
     *
     * @param pageDTO 分页条件查询DTO{@link SysOrgPageDTO}
     * @return
     */
    Page<SysOrgPageVO> page(SysOrgPageDTO pageDTO);
}

```

## ServiceImpl

```java
package com.crunii.itss.service.sys.impl;

import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.toolkit.StringUtils;
import com.baomidou.mybatisplus.core.toolkit.Wrappers;
import com.crunii.common.core.entity.dto.BasicIdListDTO;
import com.crunii.common.meta.exception.BusinessException;
import com.crunii.common.mybatis.extend.CruniiLambdaQueryWrapper;
import com.crunii.common.mybatis.query.Page;
import com.crunii.common.mybatis.service.impl.CruniiServiceImpl;
import com.crunii.common.util.lang.AssertUtil;
import com.crunii.itss.entity.dto.sys.SysOrgPageDTO;
import com.crunii.itss.entity.dto.sys.SysOrgSaveDTO;
import com.crunii.itss.entity.po.sys.SysOrg;
import com.crunii.itss.entity.vo.sys.SysOrgPageVO;
import com.crunii.itss.entity.vo.sys.SysOrgTreeVO;
import com.crunii.itss.mapper.sys.SysOrgMapper;
import com.crunii.itss.service.sys.ISysOrgService;
import org.springframework.beans.BeanUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.ArrayList;
import java.util.HashMap;
import java.util.List;
import java.util.Map;

/**
 * 组织Service实现类
 *
 * @Author Liuyh
 * @Date 2023/3/13
 * @Version V0.0.1
 */
@Service
public class SysOrgServiceImpl extends CruniiServiceImpl<SysOrgMapper, SysOrg> implements ISysOrgService {
    @Autowired
    private SysOrgMapper sysOrgMapper;

    /**
     * 新增/修改组织
     *
     * @param saveDTO 组织信息 {@link SysOrgSaveDTO}
     * @return 组织信息
     */
    @Transactional(rollbackFor = Exception.class)
    @Override
    public Boolean save(SysOrgSaveDTO saveDTO) {
        // 判断编码是否重复
        LambdaQueryWrapper<SysOrg> codeWrapper = new LambdaQueryWrapper<>();
        codeWrapper.eq(SysOrg::getCode, saveDTO.getCode());
        if (StringUtils.isNotBlank(saveDTO.getId())) {
            codeWrapper.ne(SysOrg::getId, saveDTO.getId());
        }
        long count = sysOrgMapper.selectCount(codeWrapper);
        if (count > 0) {
            throw new BusinessException("组织编码已存在");
        }

        SysOrg sysOrg = new SysOrg();
        BeanUtils.copyProperties(saveDTO, sysOrg);
        boolean state = saveOrUpdateOfSort(sysOrg);
        AssertUtil.isTrue(state, "保存失败");
        return state;
    }

    /**
     * 删除组织
     *
     * @param id 组织id
     */
    @Transactional(rollbackFor = Exception.class)
    @Override
    public void deleteSysOrg(String id) {
        // 判断是否有子节点
        LambdaQueryWrapper<SysOrg> wrapper = Wrappers.lambdaQuery();
        wrapper.eq(SysOrg::getParentId, id);
        long count = sysOrgMapper.selectCount(wrapper);
        if (count > 0) {
            throw new BusinessException("该组织下存在子组织，无法删除");
        }
        removeById(id);
    }

    /**
     * 批量删除组织
     *
     * @param idListDTO 组织id集合 {@link BasicIdListDTO}
     */
    @Transactional(rollbackFor = Exception.class)
    @Override
    public void batchDeleteSysOrg(BasicIdListDTO idListDTO) {
        for (String id : idListDTO.getIds()) {
            deleteSysOrg(id);
        }
    }

    /**
     * 查询组织树
     *
     * @return 系统架构树
     */
    @Override
    public List<SysOrgTreeVO> getSysOrgTree() {
        // 查询组织列表
        List<SysOrgTreeVO> sysOrgTreeVOList = new ArrayList<>();
        LambdaQueryWrapper<SysOrg> wrapper = new LambdaQueryWrapper<>();
        wrapper.orderByAsc(SysOrg::getSortIndex);
        List<SysOrg> sysOrgList = sysOrgMapper.selectList(wrapper);
        // 将组织列表转换为组织树
        Map<String, SysOrgTreeVO> sysOrgVOMap = new HashMap<>();
        for (SysOrg sysOrg : sysOrgList) {
            SysOrgTreeVO sysOrgTreeVO = new SysOrgTreeVO();
            BeanUtils.copyProperties(sysOrg, sysOrgTreeVO);
            sysOrgVOMap.put(sysOrg.getId(), sysOrgTreeVO);
        }
        for (SysOrg sysOrg : sysOrgList) {
            SysOrgTreeVO sysOrgTreeVO = sysOrgVOMap.get(sysOrg.getId());
            if (StringUtils.isNotBlank(sysOrg.getParentId())) {
                SysOrgTreeVO parentVO = sysOrgVOMap.get(sysOrg.getParentId());
                if (parentVO != null) {
                    if (parentVO.getChildren() == null) {
                        parentVO.setChildren(new ArrayList<>());
                    }
                    sysOrgTreeVO.setParentName(parentVO.getName());
                    parentVO.getChildren().add(sysOrgTreeVO);
                }
            } else {
                sysOrgTreeVOList.add(sysOrgTreeVO);
            }
        }
        return sysOrgTreeVOList;
    }

    /**
     * 分页条件查询组织列表
     *
     * @param pageDTO 分页条件查询DTO{@link SysOrgPageDTO}
     * @return
     */
    @Override
    public Page<SysOrgPageVO> page(SysOrgPageDTO pageDTO) {
        CruniiLambdaQueryWrapper<SysOrg> queryWrapper = CruniiLambdaQueryWrapper.wrapper();
        queryWrapper.conditionLike(SysOrg::getName, pageDTO.getName());
        queryWrapper.conditionLike(SysOrg::getCode, pageDTO.getCode());
        Page<SysOrgPageVO> page = pageToVo(pageDTO, queryWrapper, SysOrgPageVO.class);
        return page;
    }
}
```

代码模板：Controller

```java
package com.crunii.itss.api.controller.sys;


import com.crunii.common.core.entity.dto.BasicIdListDTO;
import com.crunii.common.core.response.Result;
import com.crunii.common.mybatis.query.Page;
import com.crunii.itss.entity.dto.sys.SysOrgPageDTO;
import com.crunii.itss.entity.dto.sys.SysOrgSaveDTO;
import com.crunii.itss.entity.vo.sys.SysOrgPageVO;
import com.crunii.itss.entity.vo.sys.SysOrgTreeVO;
import com.crunii.itss.service.sys.ISysOrgService;
import io.swagger.annotations.Api;
import io.swagger.annotations.ApiOperation;
import org.springframework.validation.annotation.Validated;
import org.springframework.web.bind.annotation.*;

import javax.annotation.Resource;
import java.util.List;

/**
 * 组织控制器
 * @author liuyh
 */
@RestController
@RequestMapping("/sys/org")
@Api(tags = "组织管理")
public class SysOrgController {
    @Resource
    private ISysOrgService sysOrgService;

    /**
     * 新增/修改组织
     *
     * @param sysOrgSaveDTO 组织信息{@link SysOrgSaveDTO}
     * @return 组织信息
     */
    @PostMapping("/save")
    @ApiOperation(value = "新增/修改组织")
    public Result save(@RequestBody @Validated SysOrgSaveDTO sysOrgSaveDTO) {
        sysOrgService.save(sysOrgSaveDTO);
        return Result.ok();
    }

    /**
     * 批量删除组织
     *
     * @param idListDTO 组织id集合 {@link BasicIdListDTO}
     */
    @PostMapping("/remove")
    @ApiOperation(value = "批量删除组织")
    public Result remove(@Validated @RequestBody BasicIdListDTO idListDTO) {
        sysOrgService.batchDeleteSysOrg(idListDTO);
        return Result.ok();
    }

    /**
     * 查询组织树
     *
     * @return 系统架构树
     */
    @GetMapping("/tree")
    @ApiOperation(value = "查询组织树")
    public Result<List<SysOrgTreeVO>> getSysOrgTree() {
        return Result.ok(sysOrgService.getSysOrgTree());
    }

    /**
     * 分页条件查询组织列表
     *
     * @param pageDTO 分页条件查询DTO{@link SysOrgPageDTO}
     * @return
     */
    @PostMapping("/page")
    @ApiOperation(value = "分页条件查询组织列表")
    public Result<Page<SysOrgPageVO>> page(@RequestBody SysOrgPageDTO pageDTO) {
        return Result.ok(sysOrgService.page(pageDTO));
    }
}
```

## SaveDTO

```java
package com.crunii.itss.entity.dto.sys;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;
import org.hibernate.validator.constraints.Length;

import javax.validation.constraints.NotBlank;
import javax.validation.constraints.NotNull;

/**
 * 组织保存DTO
 * @author liuyh
 */
@Data
@ApiModel(description = "组织保存DTO")
public class SysOrgSaveDTO {
    /**
     * 组织id
     */
    @ApiModelProperty(value = "组织id(不为空是更新组织)")
    private String id;

    /**
     * 组织名称
     */
    @ApiModelProperty(value = "父级组织id")
    private String parentId;

    /**
     * 组织名称
     */
    @NotBlank(message = "组织名称不能为空")
    @Length(max = 50, message = "组织名称长度不能超过50")
    @ApiModelProperty(value = "组织名称")
    private String name;

    /**
     * 组织类型
     */
    @NotNull(message = "组织类型不能为空")
    @ApiModelProperty(value = "组织类型")
    private Integer type;

    /**
     * 组织编码
     */
    @NotBlank(message = "组织编码不能为空")
    @Length(max = 20, message = "组织编码长度不能超过20")
    @ApiModelProperty(value = "组织编码")
    private String code;

    /**
     * 排序索引
     */
    @ApiModelProperty(value = "排序索引")
    private Integer sortIndex;
}
```

## PageDTO

```java
package com.crunii.itss.entity.dto.sys;

import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.core.metadata.OrderItem;
import com.crunii.common.core.entity.dto.AbstractPageDTO;
import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

import java.util.List;

/**
 * 组织分页查询DTO
 *
 * @Author Liuyh
 * @Date 2023/3/23
 * @Version V0.0.1
 */
@ApiModel("组织分页查询DTO")
@Data
public class SysOrgPageDTO extends AbstractPageDTO {
    /**
     * 角色编码
     */
    @ApiModelProperty("角色编码")
    private String code;

    /**
     * 角色名称
     */
    @ApiModelProperty("角色名称")
    private String name;
}
```

## PageVO

```java
package com.crunii.itss.entity.vo.sys;

import io.swagger.annotations.ApiModel;
import io.swagger.annotations.ApiModelProperty;
import lombok.Data;

/**
 * 组织架构分页数据VO
 *
 * @Author Liuyh
 * @Date 2023/3/23
 * @Version V0.0.1
 */
@ApiModel("组织架构分页数据VO")
@Data
public class SysOrgPageVO {
    /**
     * 组织id
     */
    @ApiModelProperty(value = "组织id")
    private String id;

    /**
     * 父级组织id
     */
    @ApiModelProperty(value = "父级组织id")
    private String parentId;

    /**
     * 父级组织名称
     */
    @ApiModelProperty(value = "父级组织名称")
    private String parentName;

    /**
     * 组织名称
     */
    @ApiModelProperty(value = "组织名称")
    private String name;

    /**
     * 组织类型
     */
    @ApiModelProperty(value = "组织类型")
    private Integer type;

    /**
     * 组织编码
     */
    @ApiModelProperty(value = "组织编码")
    private String code;

    /**
     * 序号
     */
    @ApiModelProperty(value = "序号")
    private Integer sortIndex;
}
```


